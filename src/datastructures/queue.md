# Queue in Csharp

## Implementation
```cs
public static class Example
{
    public static void Run() 
    {
        Console.WriteLine("--- Queue Example: ---");
        var list = new QueueUsingLinkedList<string>();
        list.Add("Hello");
        list.Add("World");
        list.Add("how's");
        list.Add("it");
        list.Add("going?");
        Console.WriteLine(list);

        Console.WriteLine(list.Remove());
            
        foreach(var node in list) Console.WriteLine(node);
    }
}
    
/// Queue implementation, First in First Out based on the Default Linked List.
public sealed class QueueUsingLinkedList<T> :IEnumerable<T> where T : IComparable<T>
{
    private LinkedList<T> list = new LinkedList<T>();
        
    public int Length => list.Count;

    public bool IsEmpty() => list.Count == 0;

    public T? PeekFirst() => list.First is null ? default(T) : list.First.Value;    // the default(T) should never get reached

    public T? PeekLast() => list.Last is null ? default(T) : list.Last.Value;

    /// Enqueue - Adds at the End of the Queue
    public void Add(T value) => list.AddLast(value);

    /// Dequeue - Removes from the Top of the Queue
    public T? Remove() {
        if (this.IsEmpty()) throw new InvalidOperationException("Cant Remove from empty Queue");
        var result = PeekFirst();
        list.RemoveFirst();
        return result;
    }

    public override string ToString() {
        return "[Queue: out< " + string.Join(", ", this) +" <in]";
    }

    public IEnumerator<T> GetEnumerator() => list.GetEnumerator();

    IEnumerator IEnumerable.GetEnumerator() => list.GetEnumerator();
}
```
