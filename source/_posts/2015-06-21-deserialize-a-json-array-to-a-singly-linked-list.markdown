---
layout: post
title: "Deserialize a JSON array to a singly linked list"
date: 2015-06-21 21:19:53 -0700
comments: true
categories: [Java, Jackson, JSON]
---

We know that Jackson is very convenient to deserialize a JSON string into a `ArrayList`, `HashMap` or POJO object. 

But how to deserialize a JSON array, such as `[1,2,3,4,5]` to a singly linked list?

The definition of singly linked list is as follows:

```java
public class SinglyLinkedListNode<E>
{
    public E value;
    public SinglyLinkedListNode<E> next;
}
```

Well, the solution is quite simpler than I expected:  Just make `SinglyLinkedListNode` implement `java.util.List` or `java.util.Collection` , Jackson will automatically deserialize it!

This idea comes from [Tatu Saloranta](https://github.com/cowtowncoder), the discussion post is [here](https://groups.google.com/forum/#!topic/jackson-user/EbltfyXSVV8), special thanks to him!

Here is the complete code of `SinglyLinkedListNode`:

<!-- more -->

```java
package org.algorithmhub.customized_collection;

import java.util.*;

/**
 * Singly Linked List.
 *
 * <p>As to singly linked list, a node can be viewed as a single node,
 * and it can be viewed as a list too.</p>
 *
 * @param <E> the type of elements held in this collection
 * @see java.util.LinkedList
 */
public class SinglyLinkedListNode<E>
    extends AbstractSequentialList<E>
    implements Cloneable, java.io.Serializable
{
    public E value;
    public SinglyLinkedListNode<E> next;

    /**
     * Constructs an empty list.
     */
    public SinglyLinkedListNode() {
        value = null;
        next = null;
    }

    /**
     * Constructs an list with one element.
     */
    public SinglyLinkedListNode(final E value, SinglyLinkedListNode<E> next) {
        this.value = value;
        this.next = next;
    }

    /**
     * Constructs a list containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     *
     * @param  c the collection whose elements are to be placed into this list
     * @throws NullPointerException if the specified collection is null
     */
    public SinglyLinkedListNode(Collection<? extends E> c) {
        this();
        addAll(c);
    }

    /**
     * @inheritDoc
     */
    public int size() {
        int size = 0;
        if (value == null) return size;

        SinglyLinkedListNode<E> cur = this;
        while (cur != null) {
            size++;
            cur = cur.next;
        }
        return size;
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
        return "Index: "+index+", Size: " + size();
    }

    private void checkPositionIndex(int index) {
        if (!isPositionIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    private class PositionInfo {
        private SinglyLinkedListNode<E> prev;
        private SinglyLinkedListNode<E> cur;
        private PositionInfo(SinglyLinkedListNode<E> prev, SinglyLinkedListNode<E> cur) {
            this.prev = prev;
            this.cur = cur;
        }
    }
    /**
     * Returns the (non-null) Node at the specified element index.
     */
    PositionInfo node(int index) {
        checkPositionIndex(index);
        if (this.value == null) return new PositionInfo(null, null);

        SinglyLinkedListNode<E> prev = new SinglyLinkedListNode<E>(null, this);
        SinglyLinkedListNode<E> cur = this;
        for (int i = 0; i < index; i++) {
            prev = prev.next;
            cur = cur.next;
        }
        return new PositionInfo(prev, cur);
    }

    /**
     * @inheritDoc
     */
    public ListIterator<E> listIterator(int index) {
        checkPositionIndex(index);
        return new ListItr(index);
    }

    private class ListItr implements ListIterator<E> {
        private SinglyLinkedListNode<E> prev;
        private SinglyLinkedListNode<E> cur;
        private int expectedModCount = modCount;

        ListItr(int index) {
            assert isPositionIndex(index);
            final PositionInfo positionInfo = node(index);
            prev = positionInfo.prev;
            cur = positionInfo.cur;
        }

        public boolean hasNext() {
            return cur != null;
        }

        public E next() {
            checkForComodification();
            if (!hasNext())
                throw new NoSuchElementException();

            final E result = cur.value;
            cur = cur.next;
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

        public void add(E e) {
            checkForComodification();
            if (prev == null) { // empty list
                SinglyLinkedListNode.this.value = e;
                SinglyLinkedListNode.this.next = null;
                cur = SinglyLinkedListNode.this;
            } else if (cur == null) {  // linkLast
                prev.next = new SinglyLinkedListNode<>(e, null);
            } else {
                final SinglyLinkedListNode<E> newNode = new SinglyLinkedListNode<>(e, cur.next);
                cur.next = newNode;
            }
            modCount++;
            expectedModCount++;
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
}
```

Then comes with the unit tests:

```java
package org.algorithmhub.customized_collection;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.Test;
import static org.junit.Assert.assertEquals;

import java.io.IOException;
import java.util.ArrayList;
import java.util.Arrays;

public class SinglyLinkedListTest {

    @Test public void singlyLinkedListTest() throws IOException {
        final ObjectMapper objectMapper = new ObjectMapper();

        final SinglyLinkedListNode<Integer> emptyList = objectMapper.readValue("[]",
            new TypeReference<SinglyLinkedListNode<Integer>>() {});
        final SinglyLinkedListNode<Integer> emptyListExpected = new SinglyLinkedListNode<>();
        assertEquals(emptyListExpected, emptyList);

        // the time complexity is O(n)
        final SinglyLinkedListNode<Integer> intList = objectMapper.readValue("[1,2,3,4,5]",
            new TypeReference<SinglyLinkedListNode<Integer>>() {});
        final SinglyLinkedListNode<Integer> intListExpected = new SinglyLinkedListNode<>(1,
            new SinglyLinkedListNode<>(2,
                new SinglyLinkedListNode<>(3,
                    new SinglyLinkedListNode<>(4,
                        new SinglyLinkedListNode<>(5, null)))));
        intListExpected.next = new SinglyLinkedListNode<>(2,
            new SinglyLinkedListNode<>(3,
                new SinglyLinkedListNode<>(4,
                    new SinglyLinkedListNode<>(5, null))));
        assertEquals(intListExpected, intList);

        final ArrayList<SinglyLinkedListNode<Integer>> arrayOfList = objectMapper.readValue("[[1,2,3], [4,5,6]]",
            new TypeReference<ArrayList<SinglyLinkedListNode<Integer>>>() {});
        final ArrayList<SinglyLinkedListNode<Integer>> arrayOfListExpected = new ArrayList(Arrays.asList(
            new SinglyLinkedListNode<>(Arrays.asList(1, 2, 3)),
            new SinglyLinkedListNode<>(Arrays.asList(4, 5, 6))));
        assertEquals(arrayOfListExpected, arrayOfList);
    }
}
```


## Related Posts

[Deserialize a JSON String to a Binary Tree](http://www.soulmachine.me/blog/2015/06/22/deserialize-a-json-string-to-a-binary-tree/)
