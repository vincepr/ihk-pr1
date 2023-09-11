# Priority Queue - (often implemented with a Heap)
Similar to normal queue BUT items of higher priority come out first.
- items need to be comparable for this. So the comparable data must be able to be sorted in some way.

## Heap (often used for priority queue)
a tree that satisfies the heap property. `If A is a parent of node B, then A is ordered with respect to B for all nodes A,B in the heap`
- Ex. if A the parent then all children and it's children are smaller.

## Usage
- can be used to implement Dijkstra's Shortest Path Algorithm
- if we for example always want to the next best (or next worst) node
    - Best First Search (BFS) often grab the next most promising node like this
- used in Huffman coding (lossless data compression)
- Minimum Spaning Tree algorithms (MST)

## Complexity
| | |
|---|---|
|Construction| O(n) |
| Polling | O(log(n)) |
| Peeking | O(1) |
| Adding| O(log(n)) |
| Naive Removing| O(n) |
| hash-table removing | O(log(n))  |
| Naive contains | O(n) |
| hash-table contains | O(1) |
- A hashtable ontop of the Heap adds overhead but makes remove() and contains() faster

## Max-Heap Implementation (in go) 
```go
// our processes we want to queue (bigger prio -> do first)
type Process struct{
    prio int
}

// our heap structure (max heap in this case)
type Heap struct{
    arr []Process
}


// public function to add a element to the heap
func (h *Heap) Insert(proc Process){
    h.arr =  append(h.arr, proc)
    h.heapifyUp(len(h.arr)-1)
}
// bring heap back into heap-state after a Input()
// does so by swapping with parent till uptop or not bigger anymore
func (h *Heap)heapifyUp(idx int){
    for h.arr[idx].prio > h.arr[parent(idx)].prio {         // while( node>parent )
        h.swap(parent(idx), idx)
        idx = parent(idx)
    }
}


// public function to "pop()" the largest/root node
func (h *Heap) Extract() (Process, error) {
    length := len(h.arr) -1
    if length < 0 {
        return Process{}, fmt.Errorf("Heap is Empty, can not remove anything")
    }
    popElement := h.arr[0]
    h.arr[0] = h.arr[length]    // swap last element to first
    h.arr = h.arr[:length]      // remove last slice element (but does not reallocate in go if i understand correctly)

    h.heapifyDown(0)            // start our sort-shuffle from index 0
    return popElement, nil
}
// bring heap back into heap-state after a Extract()
// does so by potentially swapping with bigger child, moving down till bottom/no more swap
func (h *Heap)heapifyDown(idx int){
    current := idx
    last    := len(h.arr)-1
    l, r    := left(idx), right(idx)
    for l <= last {
        if l == last {
            current = l
        } else if h.arr[l].prio > h.arr[r].prio{
            current = l
        } else {
            current = r
        }
        if h.arr[idx].prio < h.arr[current].prio{
            h.swap(idx, current)
            idx = current
            l, r = left(idx) , right(idx)
        } else { return }
    }
}


/*
*   helpers
*/

// returns the equivalent parent/left/right node of our "thought off binary-tree"
func parent(idx int) int {
    return (idx -1) / 2
}

func left(idx int) int {
    return 2*idx +1
}

func right(idx int) int {
    return 2*idx +2
}

func (h *Heap)swap(i1 int, i2 int){
    h.arr[i1], h.arr[i2] = h.arr[i2], h.arr[i1]
}
```

## Min and Max Heap in Csharp
```cs
internal static class Example
{
    public static void Run()
    {
        {
            Console.WriteLine("--- Example: MAX PRIORITY QUEUE ---");
            var max = new MaxPriorityQueue<int, char>();

            max.Add(5, 'a');
            max.Add(9, 'b');
            max.Add(12, 'c');
            max.Add(13, 'd');
            max.Add(16, 'e');
            max.Add(45, 'f');

            foreach (var item in max) Console.WriteLine(item);

            Console.WriteLine($"Count: {max.Count}");
            for (int i = max.Count; i > 0; i--)
            {
                //var (_,value) = min.Pop();
                Console.WriteLine("Out: " + max.Pop());
                Console.WriteLine($"new Count: {max.Count}");
                //foreach (var item in min) Console.WriteLine(item);
            }
        }
        {
            Console.WriteLine("--- Example: MIN PRIORITY QUEUE ---");
            var min = new MinPriorityQueue<int, char>();

            min.Add(5, 'a');
            min.Add(9, 'b');
            min.Add(12, 'c');
            min.Add(13, 'd');
            min.Add(16, 'e');
            min.Add(45, 'f');

            foreach (var item in min) Console.WriteLine(item);

            Console.WriteLine($"Count: {min.Count}");
            for (int i = min.Count; i > 0; i--)
            {
                //var (_,value) = min.Pop();
                Console.WriteLine("Out: " + min.Pop());
                Console.WriteLine($"new Count: {min.Count}");
                //foreach (var item in min) Console.WriteLine(item);
            }
        }
    }
}

public class MaxPriorityQueue<TPriority, TValue> : IEnumerable<KeyValuePair<TPriority, TValue>>
{
    /// <summary>
    /// Even though we store it as an array-like form it is actually a Binary Tree. 
    /// </summary>
    //           0 == root,
    //          /        \
    //   1 == left     2 == right
    //  /        \        /       \
    // 3 left  4 right  5left    6 right

    private protected List<KeyValuePair<TPriority, TValue>> _heap;

    private protected IComparer<TPriority> _comparer;

    public int Count => _heap.Count;

    public MaxPriorityQueue() : this(Comparer<TPriority>.Default) { }

    public MaxPriorityQueue(IComparer<TPriority> comparer)
    {
        _heap = new List<KeyValuePair<TPriority, TValue>>();
        _comparer = comparer;
    }


    public void Add(TPriority priority, TValue value)
    {
        _heap.Add(new KeyValuePair<TPriority, TValue>(priority, value));
        HeapifyUp(_heap.Count - 1);
    }
    public void Insert(params KeyValuePair<TPriority, TValue>[] values)
    {
        foreach (var value in values) Add(value.Key, value.Value);
    }

    // bring heap back into heap-state after Insert(). does so by swapping with parent till uptop or bigger no more
    private void HeapifyUp(int idx)
    {
        while (Compare(idx, Parent(idx)))
        {
            Swap(Parent(idx), idx);
            idx = Parent(idx);
        }
    }

    public TValue? Peek() => (_heap.Count > 0) ? _heap[0].Value : default;

    // pops the value with highest priority from our Priority queue
    public (TValue? value, bool success) Pop()
    {
        int len = _heap.Count - 1;
        if (len < 0) return (default(TValue), false);

        // swap last element in place of removed first
        var value = _heap[0].Value;
        _heap[0] = _heap[len];
        _heap.RemoveAt(len);
        HeapifyDown(0);
        return (value, true);
    }

    // bring heap back into heap-state after a Pop()
    // does so by potentially swapping with bigger child, moving down till bottom/no more swap
    private void HeapifyDown(int idx)
    {
        int current = idx;
        int last = _heap.Count - 1;
        var (left, right) = (Left(idx), Right(idx));
        while (left <= last) {
            if (left == last)
                current = left;
            else if (Compare(left, right))
                current = left;
            else
                current = right;

            if (_comparer.Compare(_heap[idx].Key, _heap[current].Key) < 0)
            {
                Swap(idx, current);
                idx = current;
                (left, right) = (Left(idx), Right(idx));
            }
            else return;
        }
    }

    // helpers  - to 'translate' from array structure to binary tree representation it represents
    private static int Parent(int idx) => (idx - 1) / 2;

    private static int Left(int idx) => 2 * idx + 1;

    private static int Right(int idx) => 2 * idx + 2;

    private void Swap(int idx1, int idx2)
        => (_heap[idx1], _heap[idx2]) = (_heap[idx2], _heap[idx1]);

    private bool Compare(int idx1, int idx2)
        => _comparer.Compare(_heap[idx1].Key, _heap[idx2].Key) > 0;

    public IEnumerator<KeyValuePair<TPriority, TValue>> GetEnumerator()
        => ((IEnumerable<KeyValuePair<TPriority, TValue>>)_heap).GetEnumerator();

    IEnumerator IEnumerable.GetEnumerator()
        => ((IEnumerable)_heap).GetEnumerator();
}

public class MinPriorityQueue<TPriority, TValue> : MaxPriorityQueue<TPriority, TValue>
{
    // we just have to map -1 -> +1 && +1 -> -1 && 0 -> 0
    // and we changed our MaxPriorityQueue to a MinPriorityQueue
    // we do this by wrapping the Comparer and negating all comparisons.

    private class InverseComparer : IComparer<TPriority>
    {
        private Comparer<TPriority> _originalComparer;

        public InverseComparer(Comparer<TPriority> comparer)
        {
            this._originalComparer = comparer;
        }

        public int Compare(TPriority? x, TPriority? y)
        {
            return -_originalComparer.Compare(x, y);
        }
    }

    public MinPriorityQueue() : base(new InverseComparer(Comparer<TPriority>.Default)) { }

    public MinPriorityQueue(IComparer<TPriority> comparer)
    {
        _heap = new List<KeyValuePair<TPriority, TValue>>();
        _comparer = new InverseComparer((Comparer<TPriority>)comparer);
    }
}
```