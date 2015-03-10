% Rust Radix Tries

Michael Sproul

<img id="trie" src="images/radix-trie.png">

# What is Rust?

Rust is a low-level programming language like C++, with functional programming influences.

* Strong static typing.
* Algebraic data types and pattern matching.
* Type inference.
* Type-classes (traits) and generics.

# What is Rust?

```rust
/// Insert the given key-value pair, returning any previous value associated with the key.
pub fn insert(&mut self, key: K, value: V) -> Option<V> {
    let key_fragments = NibbleVec::from_byte_vec(key.encode());
    let result = self.root.insert(key, value, key_fragments);

    if result.is_none() {
        self.length += 1;
    }

    result
}
```

# Safer Systems Programming

* **Guaranteed** memory safety via ownership and borrowing.
    * No use after free errors.
    * No segfaults.
    * No data-races - safe multi-threading!
    * No need to release files, sockets, mutexes.

Ownership is RAII and `std::auto_ptr` perfected.

# Problems with Ownership...

In a tree-like data-structure, parents own their children...

* Children can't hold pointers to their parents like in C/Java.
* Removing a key from a Radix Trie may require deleting its parents...

# The Solution

* Use recursion liberally.
* Store actions in an abstract way so they can be performed later.

```rust
enum DeleteAction<K, V> {
    Replace(Box<TrieNode<K, V>>),
    Delete,
    DoNothing
}
```

# The Solution

```rust
// Recurse down the Trie working out what to do.
let (value, delete_action) = match self.children[bucket] {
    None => (None, DoNothing),
    Some(box ref mut existing_child) => // Do stuff...
};

// Apply the computed delete action.
match delete_action {
    Replace(node) => {
        self.children[bucket] = Some(node);
        (value, DoNothing)
    }
    // More cases...
}
```

# Benchmarks

Rust does well against Go and Haskell.

<table class="benchmark">
<tr>
<th># Words</th>
<th>Rust</th>
<th>Haskell</th>
<th>Go</th>
</tr>
<tr>
<td>226</td>
<td>0.14 ms</td>
<td class="winner">0.11 ms</td>
<td>0.29 ms</td>
</tr>
<tr>
<td>103,366</td>
<td>90 ms</td>
<td>413 ms</td>
<td class="winner">60 ms</td>
</tr>
</table>

<small>Shown: time for all words to be inserted.</small>

# Thank you!

<div align="center">
<img id="rust-logo" src="http://www.rust-lang.org/logos/rust-logo-blk.svg"><br>
<a href="http://rust-lang.org/">rust-lang.org</a><br>
<a href="https://github.com/michaelsproul/rust_radix_trie/">github.com/michaelsproul/rust_radix_trie</a>
</div>
