---
layout:     post
title:      Idiomatic tree structures in Rust
date:       2016-12-20
summary:    A short introduction in mutable graph like structures
categories: architecture
---

If you want to create data structures which can be modified during runtime, a possible solution could lead into tree or
graph like structures. Writing tree structures in Rust is no trivial problem. Nevertheless there are some common
idiomatic ways how to handle lifetime and borrowing issues. In other languages like C/C++ we would simply use pointers
to create graphs or trees. This is also possible in Rust, but the thing is that this would kill every benefit from the
borrow checker and could lead into the same pitfalls like in other languages.

I tried a lots of implementations of tree like structures, which should be dynamically modifiable during runtime. One of
the first implementations used interior mutability with data structures like:

```rust
use std::rc::Rc;
use std::cell::RefCell;

struct Node<T> {
    previous: Rc<RefCell<Box<Node<T>>>>,
    //        ^  ^       ^   ^
    //        |  |       |   |
    //        |  |       |   - The next Node with generic `T`
    //        |  |       |
    //        |  |       - Heap allocated memory, needed
    //        |  |         if `T` is a trait object.
    //        |  |
    //        |  - A mutable memory location with
    //        |    dynamically checked borrow rules.
    //        |    Needed because `Box` is immutable.
    //        |
    //        - Reference counted pointer, will be
    //          dropped when every reference is gone.
    //          Needed to create multiple node references.

    next: Vec<Rc<RefCell<Box<T>>>>,
    data: T,
    // ...
}
```

This is one one hand hard to understand and will on the other hand lead into runtime borrow checks, which are not that
idiomatic Rust. Furthermore back references are simply not possible with this approach, since cyclic references are not
allowed within Rust. The main reason is that cyclic references could lead very fast into memory leaks if we are not
really careful. Also, this approach is not multi threading capable, but we could use an `Arc` instead of `Rc`, which will
decrease the overall performance.

To reduce the complexity of the problem we need to reduce the lifetime complexity within the structures. A good
approach is to use a [memory arena](https://en.wikipedia.org/wiki/Region-based_memory_management) for the Nodes, because
this implies that every element within the arena has the same lifetime. How would such an arena look like? What about a
simple vector:

```rust
pub struct Arena<T> {
    nodes: Vec<Node<T>>,
}
```

A node could then look like this:
```rust
pub struct Node<T> {
    parent: Option<NodeId>,
    previous_sibling: Option<NodeId>,
    next_sibling: Option<NodeId>,
    first_child: Option<NodeId>,
    last_child: Option<NodeId>,

    /// The actual data which will be stored within the tree
    pub data: T,
}

pub struct NodeId {
    index: usize,
}
```

This means we store the actual `Node` within the `Vec`, but for creating the tree, we simply use the indexes from the
vector. To create a new node we can use this method:

```rust
pub fn new_node(&mut self, data: T) -> NodeId {
    // Get the next free index
    let next_index = self.nodes.len();

    // Push the node into the arena
    self.nodes.push(Node {
        parent: None,
        first_child: None,
        last_child: None,
        previous_sibling: None,
        next_sibling: None,
        data: data,
    });

    // Return the node identifier
    NodeId { index: next_index }
}
```

This approach makes creating graph like structures very easy, where they can contain any data. A general multi
processing is also possible since parts of a vector can shared across threads safely.

To try it out, simple use the [indextree crate](https://github.com/saschagrunert/indextree) and create graph like
structures for nearly everything.
