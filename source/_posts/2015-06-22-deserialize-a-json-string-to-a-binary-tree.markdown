---
layout: post
title: "Deserialize a JSON String to a Binary Tree"
date: 2015-06-22 01:36:02 -0700
comments: true
categories: [Java, Jackson, JSON]
---
## A simple solution

We know that Jackson is very convenient to deserialize a JSON string into a `ArrayList`, `HashMap` or POJO object. 

But how to deserialize a JSON String to a binary tree?

The definition of binary tree node is as follows:

```java
public class BinaryTreeNode<E>
{
    public E value;
    public BinaryTreeNode<E> left;
    public BinaryTreeNode<E> right;
```

The JSON String is as follows:

    {
       "value": 2,
       "left": {
        "value": 1,
        "left": null,
        "right": null
      },
      "right": {
        "value": 10,
        "left": {
          "value": 5,
          "left": null,
          "right": null
        },
        "right": null
      }
    }

The above JSON string represents a binary tree as the following:

      2
     / \
    1   10
       /
      5

The solution is quite simpler, since JSON can express tree naturally, Jackson can deal with recursive tree directly. Just annotate a constructor with `JsonCreator`:

```java
public static class BinaryTreeNode<E> 
{
    public E value;
    public BinaryTreeNode left;
    public BinaryTreeNode right;

    @JsonCreator
    public BinaryTreeNode(@JsonProperty("value") final E value) {
        this.value = value;
    }
}
```

Let's write a unite test to try it:

<!-- more -->

```java
package me.soulmachine.customized_collection;

import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.Test;

import java.io.IOException;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import static org.junit.Assert.assertEquals;


public class BinaryTreeNodeTest {
    public static class BinaryTreeNode<E> {
        public E value;
        public BinaryTreeNode left;
        public BinaryTreeNode right;

        @JsonCreator
        public BinaryTreeNode(@JsonProperty("value") final E value) {
            this.value = value;
        }

        ArrayList<E> preOrder() {
            final ArrayList<E> result = new ArrayList<>();
            if (this.value == null) {
                return result;
            }
            preOrder(this, result);
            return result;
        }

        private static <E> void preOrder(BinaryTreeNode<E> root, ArrayList<E> result) {
            if (root == null)
                return;

            result.add(root.value);
            if (root.left != null)
                preOrder(root.left, result);
            if (root.right != null)
                preOrder(root.right, result);
        }
    }
    @Test public void binaryTreeNodeTest() throws IOException {
        ObjectMapper objectMapper = new ObjectMapper();
        /*
             2
            / \
           1   10
              /
             5
        */
        final String jsonStr = "{\n"
            + "  \"value\": 2,\n"
            + "  \"left\": {\n"
            + "    \"value\": 1,\n"
            + "    \"left\": null,\n"
            + "    \"right\": null\n"
            + "  },\n" + "  \"right\": {\n"
            + "    \"value\": 10,\n"
            + "    \"left\": {\n"
            + "      \"value\": 5,\n"
            + "      \"left\": null,\n"
            + "      \"right\": null\n"
            + "    },\n"
            + "    \"right\": null\n"
            + "  }\n"
            + "}";
        System.out.println(jsonStr);
        final BinaryTreeNode<Integer> intTree = objectMapper.readValue(jsonStr,
            new TypeReference<BinaryTreeNode<Integer>>() {});
        final List<Integer> listExpected = Arrays.asList(2, 1, 10, 5);
        assertEquals(listExpected, intTree.preOrder());
    }
}
```

## A compact serialization format

Well, there is a little problem here, the JSON string above is very verbose. Let's use another kind of serialization format, i.e., serialize a binary tree in level order traversal. For example, the binary tree above can be serialized as the following JSON string:

    [2,1,10,null,null,5] 

Now how to deserialize this JSON string to a binary tree?

The idea is very similar to my previous article, [Deserialize a JSON Array to a Singly Linked List](http://www.soulmachine.me/blog/2015/06/21/deserialize-a-json-array-to-a-singly-linked-list/). Just make the `BinaryTreeNode` implement `java.util.list`,  pretend that it's a list, write our own deserialization code so that Jackson can treat a binary tree as a list.

The complete code of `BinaryTreeNode` is as the following:

```java
package me.soulmachine.customized_collection;


import java.util.*;

public class BinaryTreeNode<E> extends AbstractSequentialList<E>
    implements Cloneable, java.io.Serializable {
    public E value;
    public BinaryTreeNode<E> left;
    public BinaryTreeNode<E> right;

    /** has a left child, but it's a null node. */
    private transient boolean leftIsNull;
    /** has a right child, but it's a null node. */
    private transient boolean rightIsNull;

    /**
     * Constructs an empty binary tree.
     */
    public BinaryTreeNode() {
        value = null;
        left = null;
        right = null;
    }

    /**
     * Constructs an binary tree with one element.
     */
    public BinaryTreeNode(final E value) {
        if (value == null) throw new IllegalArgumentException("null value");
        this.value = value;
        left = null;
        right = null;
    }

    /**
     * Constructs a binary tree containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     *
     * @param  c the collection whose elements are to be placed into this binary tree
     * @throws NullPointerException if the specified collection is null
     */
    public BinaryTreeNode(Collection<? extends E> c) {
        this();
        addAll(c);
    }

    /**
     * @inheritDoc
     *
     * <p>Note: null in the middle counts, so that each father in the binary tree has a
     * one-to-one mapping with the JSON array.</p>
     */
    public int size() {
        if (value == null) return 0;

        Queue<BinaryTreeNode<E>> queue = new LinkedList<>();
        queue.add(this);

        int count = 0;
        while (!queue.isEmpty()) {
            final BinaryTreeNode<E> node = queue.remove();
            ++count;
            if (node.left != null) {
                queue.add(node.left);
            } else {
                if (node.leftIsNull) ++count;
            }
            if (node.right != null) {
                queue.add(node.right);
            } else {
                if (node.rightIsNull) ++count;
            }
        }
        return count;
    }

    /**
     * Tells if the argument is the index of a valid position for an
     * iterator or an add operation.
     */
    private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size();
    }

    /**
     * Constructs an IndexOutOfBoundsException detail message.
     * Of the many possible refactorings of the error handling code,
     * this "outlining" performs best with both server and client VMs.
     */
    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+ size();
    }

    private void checkPositionIndex(int index) {
        if (!isPositionIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private class NodeAndFather {
        private BinaryTreeNode<E> node;
        private BinaryTreeNode<E> father;
        private boolean isRight;  // the father is the right child of the father

        private NodeAndFather(BinaryTreeNode<E> node, BinaryTreeNode<E> father, boolean isRight) {
            this.node = node;
            this.father = father;
            this.isRight = isRight;
        }
    }

    /**
     * Returns the (may be null) Node at the specified element index.
     */
    NodeAndFather node(int index) {
        checkPositionIndex(index);
        if (value == null) return null;

        Queue<NodeAndFather> queue = new LinkedList<>();
        queue.add(new NodeAndFather(this, null, false));

        for (int i = 0; !queue.isEmpty(); ++i) {
            final NodeAndFather nodeAndFather = queue.remove();
            if ( i == index) {
                return nodeAndFather;
            }

            if (nodeAndFather.node != null) {
                queue.add(new NodeAndFather(nodeAndFather.node.left, nodeAndFather.node, false));
                queue.add(new NodeAndFather(nodeAndFather.node.right, nodeAndFather.node, true));
            }
        }
        throw new IllegalArgumentException("Illegal index: " + index);
    }

    /**
     * @inheritDoc
     */
    public ListIterator<E> listIterator(int index) {
        checkPositionIndex(index);
        return new ListItr(index);
    }

    private class ListItr implements ListIterator<E> {
        private NodeAndFather next;
        private int nextIndex;
        private int expectedModCount = modCount;

        ListItr(int index) {
            assert isPositionIndex(index);
            next = node(index);
            nextIndex = index;
        }

        public boolean hasNext() {
            final BinaryTreeNode<E> cur = next.node;
            return cur != null || (next.father.leftIsNull || next.father.rightIsNull);
        }

        //O(n)
        public E next() {
            checkForComodification();
            if (!hasNext())
                throw new NoSuchElementException();

            final E result = next.node != null ? next.node.value : null;
            next = node(nextIndex+1);
            nextIndex++;
            return result;
        }

        public boolean hasPrevious() {
            throw new UnsupportedOperationException();
        }

        public E previous() {
            throw new UnsupportedOperationException();
        }

        public int nextIndex() {
            throw new UnsupportedOperationException();
        }

        public int previousIndex() {
            throw new UnsupportedOperationException();
        }

        public void remove() {
            throw new UnsupportedOperationException();
        }

        public void set(E e) {
            throw new UnsupportedOperationException();
        }

        public void add(E e) {  // always append at the tail
            checkForComodification();

            if (next == null) { // empty list
                BinaryTreeNode.this.value = e;
                BinaryTreeNode.this.left = null;
                BinaryTreeNode.this.right = null;
            } else {
                final BinaryTreeNode<E> newNode = e != null ? new BinaryTreeNode<>(e) : null;
                if (next.father == null) { // root
                    BinaryTreeNode<E> cur = next.node;
                    cur.left = newNode;
                    assert cur.right == null;
                    throw new UnsupportedOperationException();
                } else {
                    if (next.isRight) {
                        if (next.father.right != null) throw new IllegalStateException();
                        next.father.right = newNode;
                        if (newNode == null) {
                            next.father.rightIsNull = true;
                        }
                    } else {
                        if (next.father.left != null) throw new IllegalStateException();
                        next.father.left = newNode;
                        if (newNode == null) {
                            next.father.leftIsNull = true;
                        }

                    }
                }
            }
            modCount++;
            expectedModCount++;
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

    // the following functions are just for unit tests.
    ArrayList<E> preOrder() {
        final ArrayList<E> result = new ArrayList<>();
        if (this.value == null) {
            return result;
        }
        preOrder(this, result);
        return result;
    }

    private static <E> void preOrder(BinaryTreeNode<E> root, ArrayList<E> result) {
        if (root == null)
            return;

        result.add(root.value);
        if (root.left != null)
            preOrder(root.left, result);
        if (root.right != null)
            preOrder(root.right, result);
    }

    ArrayList<E> inOrder() {
        final ArrayList<E> result = new ArrayList<>();
        if (this.value == null) {
            return result;
        }
        inOrder(this, result);
        return result;
    }

    private static <E> void inOrder(BinaryTreeNode<E> root, ArrayList<E> result) {
        if (root == null)
            return;

        if (root.left != null)
            inOrder(root.left, result);
        result.add(root.value);
        if (root.right != null)
            inOrder(root.right, result);
    }

    ArrayList<E> postOrder() {
        final ArrayList<E> result = new ArrayList<>();
        if (this.value == null) {
            return result;
        }
        postOrder(this, result);
        return result;
    }

    private static <E> void postOrder(BinaryTreeNode<E> root, ArrayList<E> result) {
        if (root == null)
            return;

        if (root.left != null)
            postOrder(root.left, result);
        if (root.right != null)
            postOrder(root.right, result);
        result.add(root.value);
    }

    ArrayList<E> levelOrder() {
        final ArrayList<E> result = new ArrayList<>();
        if (this.value == null) {
            return result;
        }

        Queue<BinaryTreeNode<E>> queue = new LinkedList<>();
        queue.add(this);

        while (!queue.isEmpty()) {
            final BinaryTreeNode<E> node = queue.remove();
            result.add(node.value);

            if (node.left != null)
                queue.add(node.left);

            if (node.right != null)
                queue.add(node.right);
        }
        return result;
    }
}
```

Then comes with the unit tests:

```java
package me.soulmachine.customized_collection;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.Test;

import java.io.IOException;
import java.util.Arrays;
import java.util.List;

import static org.junit.Assert.assertEquals;

public class BinaryTreeNodeTest
{
    @Test public void deserializeTest() throws IOException {
        ObjectMapper objectMapper = new ObjectMapper();
        final List<Integer> intList = Arrays.asList(2,1,10,null,null,5);

        /*
             2
            / \
           1   10
              /
             5
        */
        // TODO: the time complexity is O(n^2)
        final BinaryTreeNode<Integer> intTree = objectMapper.readValue("[2,1,10,null,null,5]",
            new TypeReference<BinaryTreeNode<Integer>>() {});
        assertEquals(intList, intTree);

        assertEquals(Arrays.asList(2, 1, 10, 5), intTree.levelOrder());
        assertEquals(Arrays.asList(2, 1, 10, 5), intTree.preOrder());
        assertEquals(Arrays.asList(1, 2, 5, 10), intTree.inOrder());
        assertEquals(Arrays.asList(1, 5, 10, 2), intTree.postOrder());
    }
}
```

This article is inspired by [Tatu Saloranta](https://github.com/cowtowncoder) from [this post](https://groups.google.com/forum/#!topic/jackson-user/LzfbT0fyLWc), special thanks to him!

## Related Posts

[Deserialize a JSON Array to a Singly Linked List](http://www.soulmachine.me/blog/2015/06/21/deserialize-a-json-array-to-a-singly-linked-list/)

