# Counted-B-Trees
https://www.chiark.greenend.org.uk/~sgtatham/algorithms/cbtree.html

Introduction
B-trees are one of the best known algorithms around. A full treatment of them is available in any good algorithms book. They are a means of storing a sorted list of items, in such a way that single-item insertion, deletion and lookup all operate in log(N) time. They can be optimised for in-memory access (2-3-4 trees) or optimised to be used as an on-disk database, but the algorithms are essentially the same either way.

This page describes a small enhancement to the B-tree algorithm which allows lookups on numeric position as well as key value. Applied to an already sorted B-tree, it enables you to find arbitrary ordering statistics (median, percentiles) in log time, or to step through the tree five hundred items at a time, or other such things. It also allows you to construct an unsorted B-tree, where you specify a numeric position when inserting a new item. This structure behaves a lot like an array, except that you can insert and delete items efficiently.

(I'm not posting this idea here because I think it's new and innovative. It was so obvious to me that I'd be astonished if nobody else had ever thought of it. But a lot of people seem not to know about it, so it does qualify as unusual and not widely known.)

Algorithm Description
To recap on B-trees themselves: a B-tree is a fixed-depth tree (any path from the root to a leaf is the same length) with variable size nodes. An internal node can contain between N and 2N links to subtrees, or fewer if it's the root of the tree. Between every pair of links to subtrees is an element. (Elements are the things stored by the tree.) Leaf nodes have the same size restrictions as internal nodes, but all their subtree links are null.

Here is an example B-tree, containing the letters of the alphabet.

                             +-----------+
                             | . J . R . |
                             +-|---|---|-+
                               |   |   |
            +------------------+   |   +------------------+
            |                      |                      |
            v                      v                      v
      +-----------+            +-------+          +---------------+
      | . C . F . |            | . N . |          | . T . V . X . |
      +-|---|---|-+            +-|---|-+          +-|---|---|---|-+
        |   |   |                |   |              |   |   |   |
   +----+  ++   +---+         +--+   +--+       +---+ +-+   |   +--+
   |       |        |         |         |       |     |     |      |
   v       v        v         v         v       v     v     v      v
+-----+ +-----+ +-------+ +-------+ +-------+ +---+ +---+ +---+ +-----+
| A B | | D E | | G H I | | K L M | | O P Q | | S | | U | | W | | Y Z |
+-----+ +-----+ +-------+ +-------+ +-------+ +---+ +---+ +---+ +-----+
The change we introduce for a counted B-tree is simply this: alongside every link to a subtree, we store a count of the number of elements stored in that whole subtree. For example, this is what might happen to the tree shown above.

                             +-----------+
                             | 9 J 7 R 8 |
                             +-|---|---|-+
                               |   |   |
            +------------------+   |   +------------------+
            |                      |                      |
            v                      v                      v
      +-----------+            +-------+          +---------------+
      | 2 C 2 F 3 |            | 3 N 3 |          | 1 T 1 V 1 X 2 |
      +-|---|---|-+            +-|---|-+          +-|---|---|---|-+
        |   |   |                |   |              |   |   |   |
   +----+  ++   +---+         +--+   +--+       +---+ +-+   |   +--+
   |       |        |         |         |       |     |     |      |
   v       v        v         v         v       v     v     v      v
+-----+ +-----+ +-------+ +-------+ +-------+ +---+ +---+ +---+ +-----+
| A B | | D E | | G H I | | K L M | | O P Q | | S | | U | | W | | Y Z |
+-----+ +-----+ +-------+ +-------+ +-------+ +---+ +---+ +---+ +-----+
Then it's easy to look up the ith element in the tree: you can look at the root node and work out which subtree the element is in (or whether it's one of the elements in the root node), and then recurse down to the next node adjusting your count. In full:

We begin with i giving the index within the tree of the element we want. Let's assume that the tree is indexed from zero. We start our search at the root node.
So we're looking at a node. Let's call its subtree links S0, S1, S2, ..., and the associated element counts C0, C1, C2, ..., and the elements in between E0, E1, .... The number i denotes the index of the element we're looking for, within the subtree starting from this node. (So we adjust i every time we narrow our search to a new subtree.)
So the layout of the node is:
the subtree below S0 covers elements 0..C0-1 of the tree
element E0 is element number C0 of the tree
the subtree below S1 covers elements C0+1..C0+C1 of the tree
element E1 is element number C0+C1+1 of the tree
the subtree below S1 covers elements C0+C1+2..C0+C1+C2+1 of the tree
element E2 is element number C0+C1+C2+1 of the tree
... and so on.
So if i >= 0 and i <= C0-1, then we will descend through S0 and leave i as it is.
If i = C0, then we return E0.
If i >= 1+C0 and i <= C0+C1, then we will descend through S1, and subtract C0+1 from i.
If i = C0+C1+1, then we return E1.
... and so on. If we haven't found the node already, we will have found a node to descend to, and an adjustment of i to make it denote the index within the subtree we are descending to.
It's also easy to do an ordinary lookup by key and provide, as an extra return value, the numeric index of the element you return. Simply set a counter K to zero before beginning the lookup. Then, every time you recurse down to another node, you update K to reflect the number of elements before the beginning of the subtree you are recursing into.

The adjustments to the B-tree insertion and deletion algorithms are pretty much trivial. When you split a node into two, you must add up the subtree counts in each half to determine what to put in the subtree counts of the new links to that node. When you add or remove a subtree in the middle of a node, you need to adjust the count in the link to that node.

The insertion routine can be modified so that instead of using a sorted lookup to determine where to insert the new element, it uses the numeric lookup described above. Then you have the ability to deliberately insert an element into a B-tree at an arbitrary index. Using this, you can implement an unsorted B-tree with no comparison routine at all - inserting an item requires a given index, and lookups are only done numerically.

One further enhancement, for special purposes, might be this: if each of the elements in your tree has some notion of "size" in itself, you could store the total size of all the elements in a subtree instead of the total number of elements. This would enable a slightly different type of index lookup.

Complexity
The adjustments to existing B-tree algorithms don't change the O(log N) complexity. The new types of lookup are also O(log N). So this data structure keeps all the existing desirable properties of a B-tree and also adds new ones.

Applications
Unsorted counted B-trees are an ideal way to store the contents of an editor buffer. You have the ability to step back and forth within the structure (moving up and down), to jump to an arbitrarily numbered item (index lookup), and to insert and delete items with great efficiency. So you can do all the normal editing operations in log time. The only thing that can be done faster any other way is special cases of cut-and-paste: if your buffer were a linked list, you'd be able to unlink a section and link it in somewhere else in constant time, but with counted B-trees you have to move each item about one by one. However, the general case of cut and paste is necessarily linear-time in any case, because users often want to copy rather than cutting, and when you paste you can't just link the paste buffer into the document because the user will expect the paste buffer still to contain a copy afterwards.

Sorted counted B-trees, with the ability to look items up either by key or by number, could be useful in database-like algorithms for query planning. For example, suppose you have one set of records, indexed on two keys by two separate B-trees. Suppose you want to find all records whose first key is in one range and whose second is in another. You will have to either look up all records satisfying the first condition and then test them to see if they satisfy the second, or vice versa. But which is more efficient?

If your B-trees are counted, you can quickly find out the number of records satisfying the condition on the first key, and the number satisfying the condition on the second. Based on this, you can decide whether it would be quicker to iterate through the first index testing the second condition, or the other way round.

Finally, a counted B-tree allows you to easily extract order statistics from a changing data set: percentiles and medians. I can't think of any circumstances in which you might want to insert and remove items from a data set while keeping a running track of the median value, but if you need to do it, counted B-trees will let you.

Sample Code
A full implementation of counted 2-3-4 trees (B-trees with N=2) in C is provided for download here.

Click to download tree234.c and the header file tree234.h.

This code should be fit for production use, although of course nothing is guaranteed. It's a derivative of the ordinary (uncounted) B-tree code used in PuTTY. It's available for use under the free MIT licence: see the top of each source file for details.

It supports both sorted and unsorted trees. For sorted trees, you can look up by index or by key, and delete elements by index or by key as well. Key lookups can be inexact: for example, you can ask for the largest element that's less than or equal to x, or the smallest element greater than x, as well as just finding a specific element.

This code does not support elements with a notion of size: it only stores the count of elements.

If you define the macro TEST when compiling, it will generate a reasonably thorough stand-alone self-test.

(back to algorithms index)

(comments to anakin@pobox.com)
(thanks to chiark for hosting this page)
(last modified on Sun May 7 14:33:22 2017)
