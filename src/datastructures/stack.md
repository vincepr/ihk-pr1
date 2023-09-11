# Basic Stack implementation in Csharp


## Implementation
```cs
public static class Example{
    public static void Run() {
        Console.WriteLine("--- BasicStack Example: ---");

        var st = new BasicStack<string>(1);
        st.Push("Hello");
        st.Push("Wrong, so we remove");
        st.Pop();
        st.Push("World");
        st.Push("Whatsup");

        foreach (var item in st){
            Console.WriteLine(item.ToString());
        }

        // what happens if we iterate over empty?
        var st_empty = new BasicStack<string>(2);
        foreach (var item in st_empty){
            Console.WriteLine(item.ToString());
        }


    }
}

public sealed class BasicStack<T> : IEnumerable<T>
{
    private T[] arrayData;
    private const int defaultSize = 30;
    private int index;
    private int version;    /// this version is needed to make the Enumerator throw if accessed after change
    public BasicStack(int size = defaultSize) 
    {
        if (size <= 0) throw new ArgumentOutOfRangeException(nameof(size), "Size must be bigger than 0");
        this.arrayData = new T[size];
        this.index = 0;
        this.version = 0;
    }

    public T Pop() 
    {
        if (index == 0) throw new InvalidOperationException("Tried to remove from Empty Stack");
        index--;
        version++;
        return arrayData[index];
    }

    public void Push(T value) 
    {
        // Dynamic Array doubles in Size when full (but NEVER Shrinks!):
        if (index == arrayData.Length) {
            T[] newArr = new T[2*arrayData.Length];
            Array.Copy(arrayData, newArr, arrayData.Length);
            arrayData = newArr;
        }
        arrayData[index++] = value;
        version++;
    }

    public override string ToString(){
        return "Stack<"+string.Join(", ", arrayData)+">";
    }

    // To satisfy IEnumerable we have to provide the following 2 Methods to the iterator:
    public IEnumerator GetEnumerator()
    {
        return new Enumerator(this);
    }

    IEnumerator<T> IEnumerable<T>.GetEnumerator()
    {
        return new Enumerator(this);
    }

    /// Inner class of basic Stack - describes the Enumerator it creates
    /// that will savely without throwing iterate over it. Will loop/remove from the top first.
    /// - will throw if Original BasicStack has changed between creation and accessing it.
    ///     - this can happen in Reset() and MoveNext()
    public sealed class Enumerator : IEnumerator<T>
    {
        private BasicStack<T> stack;
        private int index;
        private int usedVersion;
        public Enumerator(BasicStack<T> stack){
            this.stack = stack;
            this.index = stack.index;
            this.usedVersion = stack.version;
        }

        /* The Enumerator implementation */

        public T Current => GetCurrent(index);


        object IEnumerator.Current => GetCurrent(index)!;

        public void Dispose()
        {
            index = -1; // no freeing or anything needed here tbh
        }

        /// true if the enumerator was successfully advanced to the next element; 
        /// false if the enumerator has passed the end of the collection.
        public bool MoveNext()
        {
            CheckVersion();
            index--;
            if (index < 0 ) return false;   // reached end
            else return true;
        }

        public void Reset()
        {
            CheckVersion();
            index = stack.index;
        }

        /* Helpers */

        private T GetCurrent(int index) 
        {
            if (index < 0) throw new InvalidOperationException("Enumerator already empty.");
            return stack.arrayData[index];
        }

        /// As Enumerator demands -> will throw if original Stack was modified and iterator is out of sync.
        private void CheckVersion(){
            if (usedVersion != stack.version) throw new InvalidOperationException("Collection modified since created.");
        }
    }
}
```