# Binary Search Tree

## Usage (all) binary Trees
- implementation of inary heaps
- Syntax trees (ex used by compiler and calculators)
- Treap - probabilistic Datastructure

## Binary Search Trees (BST)
A Special Tree, that also follows the Search Tree invariant:
- left subtree has smaller elmenets (than current node)
- right subtree has larger elements (than current node)

### Usage (Binary Search Tree)
- Implementation for Sets, Maps
- Red Black Trees
- AVL Trees
- Splay Trees

### Complexity (Binary Search Tree)
|  | Average | Worst |
|---|---|---|
| Insert | O(log(n)) | O(n) |
| Delte | O(log(n)) | O(n) |
| Remove | O(log(n)) | O(n) |
| Search | O(log(n)) | O(n) |

- Average case for statistically distributed values.
- worst case would be ex order data incoming in exactly the inverse way.

## Implementation

```cs
public class Example
{
    public static void Run()
    {
        var bst = new BST<int>(50);
        bst.Insert(99,2,234,45,34,5,91,102,46,48,47);
        //Console.WriteLine(bst.left.right.Value);
        //bst.SimplePrint();
        Console.WriteLine("~ pretty print() :");
        bst.PrettyPrint();

        Console.WriteLine("- Min: " + bst.MinValue());
        Console.WriteLine("- Max: " + bst.MaxValue());
        Console.WriteLine("- Depth: " + bst.Depth);

        Console.WriteLine("* bst.Find(101) = " + ((bst.Find(101) is null) ? "null" : bst.Find(101)));
        Console.WriteLine("* bst.Find(34) = " + ((bst.Find(34) is null) ? "null" : bst.Find(34)));

        bst.Remove(50);
        bst.Remove(34);
        bst.Remove(101);
        bst.PrettyPrint();

        Console.WriteLine("* bst.Find(34) = " + ((bst.Find(34) is null) ? "null" : bst.Find(34)));
        Console.WriteLine("---- ---- ----");

        Console.WriteLine("TraversePreOrder():");
        Console.WriteLine(String.Join(", ", bst.TraversePreOrder()));

        Console.WriteLine("TraverseInOrder():");
        Console.WriteLine(String.Join(", ", bst.TraverseInOrder()));

        Console.WriteLine("TraversePostOrder():");
        Console.WriteLine(String.Join(", ", bst.TraversePostOrder()));
    }
}

internal class BST<T> where T : IComparable
{
    public T Value { get; private set; }
    public BST<T>? left;
    public BST<T>? right;
    public int Depth => GetDepth(this) - 1;

    public BST(T value, BST<T>? left = null, BST<T>? right = null)
    {
        this.Value = value;
        this.left = left;
        this.right = right;
    }

    /* Traverse the Tree */

    public IEnumerable<T> TraversePreOrder() => TraversePreOrder(this);
    private static IEnumerable<T> TraversePreOrder(BST<T>? parent)
    {
        if (parent is not null)
        {
            yield return parent.Value;
            foreach (var item in TraversePreOrder(parent.left))
                yield return item;
            foreach (var item in TraversePreOrder(parent.right))
                yield return item;
        }
    }

    public IEnumerable<T> TraverseInOrder() => TraverseInOrder(this);
    private static IEnumerable<T> TraverseInOrder(BST<T>? parent)
    {
        if (parent is not null)
        {
            foreach (var item in TraverseInOrder(parent.left))
                yield return item;
            yield return parent.Value;
            foreach (var item in TraverseInOrder(parent.right))
                yield return item;
        }
    }

    public IEnumerable<T> TraversePostOrder() => TraversePostOrder(this);
    private static IEnumerable<T> TraversePostOrder(BST<T>? parent)
    {
        if (parent is not null)
        {
            foreach (var item in TraversePostOrder(parent.left))
                yield return item;
            foreach (var item in TraversePostOrder(parent.right))
                yield return item;
            yield return parent.Value;
        }
    }

    /* Basic operations: */
    public void Insert(params T[] values)
    {
        foreach (var val in values)
            Insert(val);
    }

    public void Insert(T newVal)
    {
        // strategy is to ignore duplicates (but some types might require more thought here (ex not every equality might be duplicate)):
        if (newVal.CompareTo(Value) == 0) { return; }
        if (newVal.CompareTo(Value) > 0)
        {
            if (right is null)
            {
                this.right = new BST<T>(newVal);
                return;
            }
            right.Insert(newVal);
        } else          // newVal < Value
        {
            if (left is null)
            {
                this.left = new BST<T>(newVal);
                return;
            }
            left.Insert(newVal);
        }
    }

    public BST<T>? Find(T value)
    {
        if (this.Value.Equals(value)) return this;
        if (value.CompareTo(this.Value) < 0)
        {
            if (left is null) return null;
            return left.Find(value);
        }
        else
        {
            if (right is null) return null;
            return right.Find(value);
        }
    }

    public T MinValue(BST<T>? node = null)
    {
        if (node is null) return MinValue(this);
        if (node.left is null) return node.Value;
        return MinValue(node.left);
    }

    public T MaxValue(BST<T>? node = null)
    {
        if (node is null) return MaxValue(this);
        if (node.right is null) return node.Value;
        return MinValue(node.right);
    }
    
    private int GetDepth(BST<T>? parent) {
        if (parent is null) return 0;
        return Math.Max( 1+GetDepth(parent.left), 1+GetDepth(parent.right));
    }

    /// <summary>
    /// deletion in BST:
    /// 1) Leaf node -> set to null
    /// 2) single child -> set child to current
    /// 3) 2 childs -> need to find inorder successor -> then copy that to current.
    /// </summary>
    /// <param name="value"></param>
    public void Remove(T value)
    {
        var res = Remove(this, value);
    }

    private static BST<T>? Remove(BST<T>? root, T value)
    {
        if (root == null) return root;  // not found / empty branch reached

        // We recursively navigate towards the target node
        if (root.Value.CompareTo(value) > 0)
        {
            root.left = Remove(root.left, value);
            return root;
        }
        else if (root.Value.CompareTo(value) < 0)
        {
            root.right = Remove(root.right, value);
            return root;
        }

        // reached target node (and only one children)
        if (root.left is null)
        {
            BST<T>? newRoot = root.right;
            root = null;
            return newRoot;
        }
        if (root.right is null)
        {
            BST<T>? newRoot = root.left;
            root = null;
            return newRoot;
        }
        // reached target node (and 2 children)
        //      find successor:
        BST<T> succParent = root;
        var succ = root.right;
        while (succ.left is not null)
        {
            succParent = succ;
            succ = succ.left;
        }
        //      Delete successor:
        if (succParent != root)
            succParent.left = succ.right;
        else
            succParent.right = succ.right;

        root.Value = succ.Value;
        succ = null;
        return root;
    }

    /* First Tree Print() implementation */
    public void PrettyPrint()
    {
        if (Value == null) return;
        Console.WriteLine(Value);
        PrintSubtree(this, "");
    }

    // helper for printTree() '+' for bigger side and '-' for smaller side
    private static void PrintSubtree(BST<T> node, String prefix)
    {
        if (node.Value == null) return;
        bool hasLeft = (node.left != null);
        bool hasRight = (node.right != null);

        if (!hasLeft && !hasRight) return;

        Console.Write(prefix);
        Console.Write((hasLeft && hasRight) ? "├─" : "");
        Console.Write((!hasLeft && hasRight) ? "└─" : "");

        if (hasRight)
        {
            bool printStrand = (hasLeft && hasRight && (node.right!.right != null || node.right.left != null));
            String newPrefix = prefix + (printStrand ? "│ " : "  ");
            Console.WriteLine("+" + node.right!.Value);
            PrintSubtree(node.right, newPrefix);
        }

        if (hasLeft)
        {
            Console.WriteLine((hasRight ? prefix : "") + "└─" + "-" + node.left!.Value);
            PrintSubtree(node.left, prefix + "  ");
        }
    }

    /* Alternative Tree Print() - more elegant but cant diff left-right */
    public IEnumerable<BST<T>> GetChildren()
    {
        if (left is not null ) yield return left;
        if (right is not null) yield return right;
    }

    public void SimplePrint(string indent="", bool isLast = true) 
    {
        Console.Write(indent);
        Console.Write(isLast ? "└─" : "├─");
        Console.Write(this.Value is null ? "null" : this.Value);
        Console.WriteLine();

        indent += isLast ? "  " : "│ ";
        var lastChild = this.GetChildren().LastOrDefault();
        foreach (var child in this.GetChildren())
            child.SimplePrint(indent, child == lastChild);
    }
}

```
