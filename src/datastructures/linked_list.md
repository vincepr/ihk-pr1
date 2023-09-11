# Linked List (Singly Linked List)


## Implementation in go
```go
package main

import "fmt"

func main() {
	// linked list example:
    var m = map[int]string{1: "2", 2: "4", 4: "8"}

    fmt.Println("keys:", MapKeys(m))

    _ = MapKeys[int, string](m)

    lst := List[int]{}
    lst.Push(10)
    lst.Push(13)
    lst.Push(23)
    fmt.Println("list:", lst.GetAll())
}

func MapKeys[K comparable, V any](m map[K]V) []K {
    r := make([]K, 0, len(m))
    for k := range m {
        r = append(r, k)
    }
    return r
}

type List[T any] struct {
    head, tail *element[T]
}

type element[T any] struct {
    next *element[T]
    val  T
}

func (lst *List[T]) Push(v T) {
    if lst.tail == nil {
        lst.head = &element[T]{val: v}
        lst.tail = lst.head
    } else {
        lst.tail.next = &element[T]{val: v}
        lst.tail = lst.tail.next
    }
}

func (lst *List[T]) GetAll() []T {
    var elems []T
    for e := lst.head; e != nil; e = e.next {
        elems = append(elems, e.val)
    }
    return elems
}
```

## Implementation in Csharp

```cs

public static class Example
{
    public static void Run() 
    {
        Console.WriteLine("--- (Singly)LinkedList Example: ---");
        var list = new SingleLinkedList<string>();

        Console.WriteLine("we expect .First on empty list to be null: "+ list.First is null);

        foreach (var item in list) Console.WriteLine("list empty -> this never gets reached"); 
        
        list.AddFirst("1");
        list.AddFirst("2");
        list.AddFirst("3");
        var node = list.First;
        node = list.GetNext(node!);
        list.AddAfter(node!, "1.5");


        foreach (var item in list) Console.WriteLine("Item is: " + item);

        list.Remove(node);

        Console.WriteLine(list.ToString());     // functionality to just print a list itself 

        list.Remove(list.FindNode("1.5"));
        Console.WriteLine(list.ToString());
    }
}

/// Singly linked list implementation.
public sealed class SingleLinkedList<T> :IEnumerable<T> where T : IComparable<T>
{
    public NodeLinkedList<T>? First {get; private set;}
    public SingleLinkedList(T[]? values = null) 
    {
        this.First = null;
        if (values != null) {
            foreach (var value in values.Reverse()) {
                this.AddFirst(value);
            }
        }
    }

    /// Adds entry to the Top of the List
    public void AddFirst(T value) 
    {
        var newNode = new NodeLinkedList<T>(value);
        newNode.Next = this.First;
        this.First = newNode;
    }

    public NodeLinkedList<T>? GetNext(NodeLinkedList<T> previous) 
    {
        if (previous == null) return null;
        return previous.Next;
    }


    public int Count => this.Aggregate(0, (acc, _)=> acc + 1);

    public void AddAfter(NodeLinkedList<T> node, T valueToAdd) 
    {
        var newNode = new NodeLinkedList<T>(valueToAdd);
        newNode.Next = node.Next;
        node.Next = newNode;
    }

    public bool Remove(NodeLinkedList<T>? node) 
    {
        var prev = this.First;
        while (prev is not null) {
            if (prev is null || prev.Next is null) return false;       // couldnt find node
            if (prev.Next == node) {
                // do steps to remove
                prev.Next = node.Next;
                return true;
            }
            // step to next node:
            prev = prev.Next;
        }
        return false;
    }

    public NodeLinkedList<T>? FindNode(T value) 
    {
        var node = this.First;
        while (node is not null) {
            if (node.Value.Equals(value)) return node;
            node = node.Next;
        }
        return null;
    }

    public override string ToString() =>
        this.Aggregate("[", (acc, x) => $"{acc} {x} ->")
            .TrimEnd('>')
            .TrimEnd('-') + "]";

    /* Enumerator to make foreach-loops, .map loops etc possible */

    public IEnumerator<T> GetEnumerator() => new Enumerator(this);

    IEnumerator IEnumerable.GetEnumerator() => new Enumerator(this);

    public sealed class Enumerator : IEnumerator<T>
    {
        private SingleLinkedList<T> originalList;
        private NodeLinkedList<T>? previous;
        private NodeLinkedList<T>? next;

        public Enumerator(SingleLinkedList<T> list) 
        {
            this.originalList = list;
            if (list.First is null) {
                // called with empty list 
                this.previous = null;
                this.next = null;
            } else {
                this.previous = null;
                this.next = list.First;
            }
        }

        // the way we check for null in MoveNext(), that gets called BEFORE each .Current
        // these should never error. But cleaner would be a custom Throw if null, just in case
        public object Current => previous!;

        T IEnumerator<T>.Current => previous!.Value;

        public void Dispose() {}                // nothing to do here really

        public bool MoveNext()
        {
            if (next is null && previous is null) return false; // started empty
            if (next is null) {
                previous = next;
                next = null;
                return false;                   // reached end after this read
            } else {
                previous = next;                // we set up next pointers
                next = previous.Next;
            }
            return true;
        }

        public void Reset()
        {
            if (originalList.First is null) {
                // called with empty list 
                this.previous = null;
                this.next = null;
            } else {
                this.previous = originalList.First;
                this.next = previous.Next;
            }
        }
    }

    /// A Single entry in our linked list is represented by this
    public sealed class NodeLinkedList<U> //: IEnumerable<T>
    {
        public NodeLinkedList(U value)
        {
            this.Value = value;
        }
        public U Value { get; }
        public NodeLinkedList<U>? Next{ get; set; }
    } 
}
```
