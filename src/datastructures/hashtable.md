# Hash Table - Hash Map
- aka HashMaps, Maps, Dictionaries etc. Key-Value pairs.
- use key (often a string) to lookup the corresponding Value.

## Strategies to handle collisions.
- Collisions are unavoidable and get more likely the higher the load-factor (aka the fuller the list / it's capacity)
- Different Strategies deal with how to handle those collisions:
### Separate Chaining
Combines a linked list with the HashTable.
- Can be extended to use Dynamic-Sized-Arrays or Self-Balancing-Trees instead of simple linked-list.

All elements that hash into the same slot index are inserted into a lined list. (ex. a .Next Attribuzte for each Entry) 
- then we can just traverse that linked list to find our Key. If we reach the end before we find it, we know its not jet in the table.
#### Advantages
- Simple to implement
- Less sensitive to hash function or load factors. (since linked list does a bunch)
- hash table doesnt need to be re-allocated since we can always just add more elements to the chain.

#### Disadvantages
- bad cache performance. Chaining is bad since not consecutive in memory. (ex. open adressing provides better chache performance)
- long chains near O(n) lookup in the worst case
- Wastage of Memory. Extra space for references. Some Parts of the table never fill up.

#### Performance


## Implementation in C - Open Adressing
For the full implementation (with context( check out https://github.com/vincepr/c_compiler/blob/main/src/table.h
- that version also uses special lookups with string interning to get even more lookup speed.

### table.h
```c
#ifndef clox_table_h
#define clox_table_h

/*
    Hash Map implementation
    - Thought: We allow variables up to eight characters long (or just hash on those first 8chars)
    - We get 26 unique chars ('a'-'Z' + '0'-'9' +'_'). We can squash all that info in a 64bit int
    - (if we map that to a continous block we would need 295148PetaByte) so we take the value modulo of the size of the array
        this folds the larger numeric range onto itself untill it fits in a smaller range of array elements
    - because of the above we have to handle hash-collisions though.
        - to mimiize the chance of collisions, we can dynamically calculate the load-factor (entry_number/bucket_number).
            if the load factor gets to big we resize and grow the array bigger
    - to avoid the 8char limit we use a deterministic/uniform/fast hash function.
        - clox implements the FNV-1a Hashing-Function   http://www.isthe.com/chongo/tech/comp/fnv/
*/

/*
    Techniques for resolving collisions: 2 basic Techniques: 
    
    SEPARATE CHAINING (not used for this):
    - each bucket containts more than one entries. For example a linked list of entries.
        To lookup an entry you just walk the linked list till you found the fitting entry
    - worst case: would be all data collides on the same bucket -> is basically one linked list
    - conceptually simple with operation implementations simple. BUT not a great fit for modern CPUs.
        Because of a lot of overhead from pointers and linked lists are scattered arround memory.
    
    OPEN ADRESSING (aka CLOSED HASHING)
    - all entries live directly in the bucket array.
    - if two entries collide in the same bucket, we find a different empty bucket to use instead.
    - the process of finding a free/available bucket is called PROBING. 
    - The Order buckets get examined is a PROBE SEQUENCE. We use a simple variant: 
        - LINEAR PROBING: IF the bucket is full we just go to the next entry and try that, then the next ...
    - Good: since it tends to cluster data its cache friendly.
*/

/*
    For notes on Tombstone strategy used, check comments for tableDelete()
*/

// The Objects our Maps Holds (ex: key:varname1, value:10.5 )
typedef struct {
    ObjString* key;
    Value value;
} Entry;

// The struct of our HashMap
typedef struct {
    int count;          // currently stores key-value pairs
    int capacity;       // capacity (so can easly get the load-factor: count/capacity)
    Entry* entries;     // array of the entries we hash
} Table;

void initTable(Table* table);
void freeTable(Table* table);
bool tableGet(Table* table, ObjString* key, Value* value);
bool tableSet(Table* table, ObjString* key, Value value);
bool tableDelete(Table* table, ObjString* key);
void tableAddAll(Table* from, Table* to);
ObjString* tableFindString(Table* table, const char* chars, int length, uint32_t hash);
void tableRemoveWhite(Table* table);
void markTable(Table* table);

/*CUSTOM:*/
bool tableFindValue(Table* table, const char* chars, int length, uint32_t hash, Value* value);


#endif
```

### table.c
```c
// if our load-factor (=entry_number/bucket_number) reaches this treshold we grow the HashMap size (reallocate it to be 2 times the size)
#define TABLE_MAX_LOAD 0.75

// constructor for the HashMap
void initTable(Table* table) {
    table->count = 0;
    table->capacity = 0;
    table->entries = NULL;
}

// basically behaves like a dynamic array (with some extra rules for inserting, delting, searching a value)
void freeTable(Table* table) {
    FREE_ARRAY(Entry, table->entries, table->capacity);
    initTable(table);
}

// HashMap-Functionality - lookup the entry in the Map
// - for-loop  keeps going bucket by bucket till it finds the key (aka probing)
// - we exit the loop if we find the key(success) or hit an Empty NULL bucket(notFound)
// - with a full HashMap the loop WOULD be infinite. BUT since we always grow it at 75% loadfactor this cant happen
// Tombstone-strategy:
//  - while probing we keep going on hitting tombstones {key:NULL, value:TRUE}
static Entry* findEntry(Entry* entries, int capacity, ObjString* key) {
    uint32_t index = key->hash & (capacity - 1);// since we dont have enough memory to map each value directly we fold our scope like this
    Entry* tombstone = NULL;                    // we store the Tombstones we hit while probing

    for (;;) {
        Entry* entry = &entries[index];
        if (entry->key == NULL) {
            if (IS_NIL(entry->value)) {         //<- empty entry
                return tombstone != NULL ? tombstone : entry;
            } else {                            //<- we found a tombstone
                if (tombstone == NULL) tombstone = entry;
            }
        } else if (entry->key == key) {
            return entry;                       //<- we found the key
        }

        index = (index + 1) & (capacity -1);    //<- modulo wraps arround if we reach the end of our capacity
    }
}

// HashMap-Functionality - If finds entry it returns true, otherwise false. 
// - value-output will point to resulting value if true
bool tableGet(Table* table, ObjString* key, Value* value) {
    if (table->count == 0) return false;
    Entry* entry = findEntry(table->entries, table->capacity, key);
    if(entry->key == NULL) return false;
    *value = entry->value;      // set the value-parameter found pointer to value
    return true;
}

// grows our HashTable size:
// we can just write over the memory (because of collisions might become less on bigger space)
// so we just make a empty new one. Then fill the table entry by entry.
// - we dont copy Tombstones -> we need to recalculate the entries-count
static void adjustCapacity(Table* table, int capacity){
    // we allocate an array of (empty) buckets
    Entry* entries = ALLOCATE(Entry, capacity);
    for (int i = 0; i<capacity; i++) {
        entries[i].key = NULL;
        entries[i].value = NIL_VAL;
    }
    table->count = 0;

    // we walk trough the old array front to back. 
    for (int i=0; i<table->capacity; i++) {
        Entry* entry = &table->entries[i];
        if (entry->key == NULL) continue;
        // We insert entries we find to the new array:
        Entry* dest = findEntry(entries, capacity, entry->key);
        dest->key = entry->key;
        dest->value = entry->value;
        table->count++;
    }
    FREE_ARRAY(Entry, table->entries, table->capacity);    // the old table can be free'd
    // we update the number of entries/capacity for the new array:
    table->entries = entries;
    table->capacity = capacity;
}

// HashMap-Functionality - Add the given key-value-pair to our table:
bool tableSet(Table* table, ObjString* key, Value value) {
    // if our load-factor gets to big (to many entries in map) we make the map bigger:
    if (table->count + 1 > table->capacity * TABLE_MAX_LOAD) {
        int capacity = GROW_CAPACITY(table->capacity);
        adjustCapacity(table, capacity);
    }
    // figure out what bucket we write to
    Entry* entry = findEntry(table->entries, table->capacity, key); 
    // then write to that bucket:
    bool isNewKey = entry->key == NULL; 
    // if key is already present we just overwrote to the same key (updated) -> we dont increment size:
    // we also check if we write to a NOT Tombstone (the nil-check) only then do we increment
    if (isNewKey && IS_NIL(entry->value)) table->count++;   
    entry->key = key;  
    entry->value = value;
    return isNewKey;
}

// HashMap-Functionality - Delete a key-value pair from the Map
// - the problem is we can just delete the entry directly. Because another entry might have dependet on it's collision when beeing entered
// - the solution is Tombstones. Instead of clearing the entry on deletion, we replace it with a special entry called tombstone
//      during probing we dont treat tombstones like empty but keep going (we treat them like full)
bool tableDelete(Table* table, ObjString* key) {
    if (table->count == 0) return false;
    Entry* entry = findEntry(table->entries, table->capacity, key);
    if (entry->key == NULL) return false;

    // place the tombstone. We use a {key: NULL, Value: true} to represent this.
    // - this is arbitrarily chosen. Any unique combination (not in use) would work.
    entry->key = NULL;
    entry->value = BOOL_VAL(true);
    return true;
}

// HashMap-Functionality - Copies all Data from one HashTable to another - ex. used for inheritance (of class-methods)
void tableAddAll(Table* from, Table* to) {
    for (int i=0; i<from->capacity; i++) {
        Entry* entry = &from->entries[i];
        if (entry->key != NULL) {
            tableSet(to, entry->key, entry->value);
        }
    }
}
```

## Separate Chaining Hash Table in Csharp

```cs

public static class Example
{
    public static void Run()
    {
        Console.WriteLine("--- (Simple) HashTable Example: ---");
        var table = new ChainingHashTable<int, int>(10)                   // setting starting capactiy =10
        {
            {5, 2},
            {23, 42},
        };

        for(int i = 10; i < 20; i++)
            table[i] = i*i*i;

        foreach (var pair in table)
            Console.WriteLine($"{pair}");


        Console.WriteLine("\n--- --- ---");
        // everytime the growth factor gets to high it reallocs with double size.
        Console.WriteLine($"Count: {table.Count} Capacity: {table.Capacity}");
        Console.WriteLine($"{table.Contains(10)} {table.GetValue(10)}");
        table.Remove(10);
        Console.WriteLine($"{table.Contains(10)} {table.GetValue(10)}");


        Console.WriteLine("\n--- --- ---");
        var table2 = new ChainingHashTable<String, int>(("Bob",24),("James",34),("Gandalf", 5000));
        table2["Gandalf"] = 55555;
        table2.Remove("Bob");
        foreach(var (key, value) in table2)
            Console.WriteLine($"<{key} {value}>");
        if (table2.Contains("James"))
            Console.WriteLine($"James is {table2["James"]}");
    }
}

/// <summary>
/// Implemention of a HashMap using SEPARATE CHAINING with Generic Key-Value pairs.
/// </summary>
/// <typeparam name="K">key</typeparam>
/// <typeparam name="V">value</typeparam>
public sealed class ChainingHashTable<K, V>  : IEnumerable<KeyValuePair<K, V>> where K : notnull
{
    /// <summary>
    /// Represents a single key-value pair in our list
    /// </summary>
    sealed class Entry
    {
        public K Key { get; init; }
        public V Value { get; set; }
        public int Hashcode { get; set; }
        public Entry? Next { get; set; }

        public Entry(K key, V value, int hashcode, Entry? next = null)
        {
            this.Key = key;
            this.Value = value;
            this.Hashcode = hashcode;
            this.Next = next;
        }
    }

    /* Attributes */
    private const int START_CAPACITY = 8;

    private Entry?[] entries;

    public int Count { get; private set; }

    /// <summary>
    /// version, so we can throw if Enumerator changed mid consuming.
    /// </summary>
    private int version = 0;                
    
    // Number of buckets
    public int Capacity => entries.Length;        

    public ChainingHashTable(int CAPACITY = START_CAPACITY)
    {
        CAPACITY = (CAPACITY > START_CAPACITY) ? CAPACITY : START_CAPACITY;
        entries = new Entry[CAPACITY];
    }

    public ChainingHashTable(params (K, V)[] pairs)
    {
        int capacity = (pairs.Length*2 > START_CAPACITY) ? pairs.Length*2 : START_CAPACITY;
        entries = new Entry[capacity];
        foreach (var pair in pairs)
            Add(pair.Item1, pair.Item2);
    }

    /* functionality METHODS */
    public void Add(K key, V value)
    {
        version++;
        int hashcode = key.GetHashCode();
        int targetBucket = (hashcode & int.MaxValue) % entries.Length;
        Entry? targetEntry = entries[targetBucket];
        
        // add directly to bucket (first element)
        if (targetEntry is null)                
        {
            entries[targetBucket] = new Entry(key, value, hashcode, null);
            Count++;
            ReallocIfNeed();
            return;
        }
        // traverse the linked list:
        while (targetEntry is not null)
        {
            // overwrite existing value
            if(key.Equals(targetEntry.Key))    
            {
                targetEntry.Value = value;
                return;
            }

            // add at end of linked list
            if (targetEntry.Next is null)       
            {
                targetEntry.Next = new Entry(key, value, hashcode, null);
                Count++;
                ReallocIfNeed();
                return;
            }
            targetEntry = targetEntry.Next;
        }
    }

    public bool Contains(K key)
    {
        var entry = FindEntry(key);
        if (entry is null) return false;
        return true;

    }

    public void Remove(K key)
    {
        version++;
        int hashcode = key.GetHashCode();
        int targetBucket = (hashcode & int.MaxValue) % entries.Length;
        Entry? previous = entries[targetBucket];
        if (previous is null) return;
        Entry? current = previous.Next;
        if (current is null) entries[targetBucket] = null;

        while (current is not null)
        {
            if (current.Hashcode == hashcode && key.Equals(current.Key))
            {
                // found Entry - so we remove it from linked list
                previous.Next = current.Next;       
                return;
            }
            previous = current;
            current = current.Next;
        }
        return;
    }

    /// <summary>
    /// returns Value stored at key or nullvalue if not found.
    /// </summary>
    public V? GetValue(K key)
    {
        var entry = FindEntry(key);
        if (entry is null) return default(V);
        return entry.Value;
    }

    /// <summary>
    /// if size reaches a certain loadfactor we reallocate with 2 times the capacity (this is slow/expensive)
    /// </summary>
    private void ReallocIfNeed()
    {
        const double LOADFACTOR = 0.75;
        const int GROWFACTOR = 2;
        if (Count < Capacity * LOADFACTOR) return;

        var newCapacity = Capacity * GROWFACTOR;
        var oldEntries = this.entries;
        this.entries = new Entry[newCapacity];
        foreach (var e in oldEntries)
        {
            if (e is not null) 
            {
                // copy roots
                this.Add(e.Key, e.Value);
                var next = e.Next;
                // copy linked elements
                while (next is not null)
                {
                    this.Add(next.Key, next.Value);
                    next = next.Next;
                }
            }
        }
    }

    private Entry? FindEntry(K key)
    {
        int hashcode = key.GetHashCode();
        int targetBucket = (hashcode & int.MaxValue) % entries.Length;
        Entry? target = entries[targetBucket];
        if (target is null) return null;
        while (target is not null)
        {
            if (target.Hashcode == hashcode && key.Equals(target.Key)) return target;
            target = target.Next;
        }
        return null;
    }

    // Inexer for our HashMap. Ex: 'var xyu = ourMap["someKey"]'
    public V? this[K key]
    {
        get => GetValue(key);
        set{ Add(key, value!); }
    }

    IEnumerator IEnumerable.GetEnumerator() => this.GetEnumerator();
    
    public IEnumerator<KeyValuePair<K, V>> GetEnumerator()
    {
        var version = this.version;
        foreach(var entry in entries)
        {
            var current = entry;
            while (current is not null)
            {
                if (this.version != version) throw new InvalidOperationException("Collection modified.");
                yield return new KeyValuePair<K, V>(current.Key, current.Value);
                current = current.Next;
            }
        }
    }
}
```
