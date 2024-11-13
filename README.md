# Iterators in Rust — The Less Known Parts

> **Source:** [Iterators in Rust — The Less Known Parts | by David Lee | in Towards Dev - Freedium](https://freedium.cfd/https://towardsdev.com/iterators-in-rust-the-less-known-parts-44cb7f7253d6)

Iterators are a powerful feature in Rust, enabling efficient and expressive iteration over collections. Recently I've been exploring the different styles of iteration and how to implement iterator support for custom collections, and it helps me understand iterator better.

## **Styles of Collection Iteration**

When it comes to iterating over collections in Rust, there are four main styles:

- **Imperative**: `for x in c { ... }` Great for when you need side effects, operations that depend on each other, or if you need to exit the loop early.
- **Functional**: `c.iter().map().filter()` Clean and concise when you just care about the results.
- **Low-level**: `c_iter.next()` Directly calls `Iterator::next()` for more control.
- **Manual**: `c.get(n)` Skips the standard iteration methods entirely.

**Opinion 💬** The functional style is usually the easiest to read, but if your `.iter()` chain gets unwieldy, feel free to use a `for` loop instead. When you're writing containers, iterator support is awesome, but if you're short on time, sometimes it's fine to just implement `.len()` and `.get()` and call it a day.Basics

**Working with Collection Iteration**

Say you have a collection `c` of type `C` that you want to iterate over. Here's a quick rundown on how to approach it:

- **`c.into_iter()`**: Turns `c` into an iterator that consumes it. This is the usual way to get an iterator if you don't need `c` afterward.
- **`c.iter()`**: Provides a borrowing iterator, leaving `c` intact.
- **`c.iter_mut()`**: Gives a mutable borrowing iterator, letting you modify `c` as you go.

**Using the Iterator**

Once you have an iterator `i`:

- **`i.next()`**: Fetches the next element, returning `Some(x)` if there's one left, or `None` when you're done.

**For Loops**

- **`for x in c {}`**: This is just shorthand for calling `c.into_iter()` and looping until there are no more elements (`None`).

**Key Points for Custom Collections**

Let's say you've made your own struct, `Collection<T> {}`. To make it iterable, here's what to add:

- **`struct IntoIter<T> {}`**: Define a struct to track the state of iteration (like an index).
- **`impl Iterator for IntoIter<T> {}`**: Implement `Iterator::next()` to return the elements one by one.

```rust
struct Collection<T> {
    // collection fields
}

struct IntoIter<T> {
    // iterator fields
}

impl<T> Iterator for IntoIter<T> {
    type Item = T;

    fn next(&mut self) -> Option<Self::Item> {
        // iteration logic
    }
}
```

At this stage, we've got an iterator-like structure, but no way to actually access it. Next let's see how to make it accessible.

## **Native Loop Support**

Most users expect our collection to work seamlessly with `for` loops. To enable this, implement the following:

- **`impl IntoIterator for Collection<T> {}`**: Allows `for x in c {}` to work.
- **`impl IntoIterator for &Collection<T> {}`**: Enables `for x in &c {}`.
- **`impl IntoIterator for &mut Collection<T> {}`**: Enables `for x in &mut c {}`.

```rust
impl<T> IntoIterator for Collection<T> {
    type Item = T;
    type IntoIter = IntoIter<T>;
     fn into_iter(self) -> Self::IntoIter {
        // logic to create an IntoIter
    }
}

impl<'a, T> IntoIterator for &'a Collection<T> {
    type Item = &'a T;
    type IntoIter = Iter<'a, T>;

    fn into_iter(self) -> Self::IntoIter {
        // logic to create an Iter
    }
}

impl<'a, T> IntoIterator for &'a mut Collection<T> {
    type Item = &'a mut T;
    type IntoIter = IterMut<'a, T>;

    fn into_iter(self) -> Self::IntoIter {
        // to create an IterMut
    }
}
```

## **Shared & Mutable Iterators**

To make our collection usable when borrowed, we'll need to add support for both shared and mutable iteration:

- **`struct Iter<T> {}`**: Define a struct to hold a `&Collection<T>` for shared (read-only) iteration.
- **`struct IterMut<T> {}`**: Define a similar struct that holds `&mut Collection<T>` for mutable (modifiable) iteration.
- **`impl Iterator for Iter<T> {}`**: Implement this to enable shared iteration.
- **`impl Iterator for IterMut<T> {}`**: Implement this to allow mutable iteration.

It's also helpful to add convenience methods:

```rust
impl<T> Collection<T> {
    fn iter(&self) -> Iter<T> {
        // logic to create an Iter
    }

    fn iter_mut(&mut self) -> IterMut<T> {
        // to create an IterMut
    }
}
```

## **Iterator Interoperability**

To enable third-party iterators to "collect into" our collection, implement:

- **`impl FromIterator for Collection<T> {}`**: This allows `some_iter.collect::<Collection<_>>()` to work seamlessly.
- **`impl Extend for Collection<T> {}`**: This enables adding elements from another iterator with `c.extend(other)`.

We might also want to add additional traits from `std::iter` to our previous iterator structs for added flexibility.

```rust
impl<T> FromIterator<T> for Collection<T> {
    fn from_iter<I: IntoIterator<Item = T>>(iter: I) -> Self {
        // logic to create a Collection from an iterator
    }
}

impl<T> Extend<T> for Collection<T> {
    fn extend<I: IntoIterator<Item = T>>(&mut self, iter: I) {
        // to extend the collection
    }
}
```

Building custom collections takes effort, but by following these steps, our collections will behave like true first-class citizens in Rust. Happy coding!