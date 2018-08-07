class: center
name: title
count: false

<img src="content/images/rust-logo-blk.svg" alt="Rust logo" width="250rem" height="auto">

# Intro to Traits

.grey[Santiago Pastorino]

.grey[.smaller[WyeWorks co-founder | Rust compiler and types team member]]

???

- Disclaimer the are a lot of concepts involved that we're going to handwave at, please take notes and investigate
- From intro - to mid - to advanced

---

<img src="content/images/rust-logo-blk.svg" alt="Rust logo" width="250rem" height="auto" style="position: absolute; right: 0rem; margin-top: -2rem;">

# Introduction

- Rust doesn’t have traditional inheritance
- How do we share behavior?
- Composition over inheritance
- Traits as a way to define shared behavior contracts.

---

# Inheritance

- Base class that defines behavior
- Other classes inherit from it to reuse that behavior and possibly override parts

```java
class Animal {
    void speak() { System.out.println("Some sound"); }
}

class Dog extends Animal {
    @Override
    void speak() { System.out.println("Woof!"); }
}
```

---

# Problems with Inheritance

- Tight Coupling
    - Subclasses are tied to the base class’s implementation and evolution.
???
- Tight Coupling: Subclasses are tied to the base class’s implementation and evolution.
--
- Fragile Base Class
    - Even small internal changes in a superclass can have unintended side effects in subclasses
???
- The Fragile Base Class Problem: Even small internal changes in a superclass can have unintended side effects in subclasses, because they inherit not only methods but also internal state and assumptions.
--
- Inflexible Hierarchies
    - Can a "FlyingFish" inherit from both Fish and Bird?
???
- Inflexible Hierarchies: Inheritance forms a tree, and each class can only have one parent (in most languages). But real-world relationships aren’t always tree-shaped.
    - In single inheritance no
    - In multiple inheritance, yes — but now you get diamond problems (ambiguous method inheritance).
--
- Inheritance Reuses Code, Not Behavior
    - class Stack extends Vector
???
- Inheritance Reuses Code, Not Behavior:
    - You often use inheritance just to “get” methods, not because the subclass is-a real specialization of the parent.
--
- Hidden State and Inheritance Hell
    - Parent classes may have hidden fields or side effects.
???
- Hidden State and Inheritance Hell: Subclasses can override methods without knowing how they interact with that hidden state.

---

# Prefer composition over inheritance

- Instead of inheriting behavior, you compose your types from smaller, independent parts.
- Each piece (component or trait) provides one clear bit of functionality.
- No base class
- You can extend behavior without touching existing code.
- Inheritance shares code. Composition shares behavior.

---

# What is a Trait?

- Traits define a set of methods that a type must implement.
- This defines a behavior contract:
    - “Any type that implements Drawable must provide a draw() method.”
- Similar to interfaces
    - can include default method implementations
    - can be implemented for types outside your crate (orphan rule).

---

# What is a Trait?

```rust
trait Drawable {
    fn draw(&self);
}
```

or ...

```rust
trait Drawable {
    fn draw(&self) {
        println!("Drawing a circle from default method");
    }
}
```

---

# Traits impls

```rust
struct Circle {
    radius: f64,
}
struct Rectangle {
    width: f64,
    height: f64,
}
impl Drawable for Circle {
    fn draw(&self) {
        println!("Drawing a circle with radius {}", self.radius);
    }
}
impl Drawable for Rectangle {
    fn draw(&self) {
        println!("Drawing a rectangle of {}x{}", self.width, self.height);
    }
}
```

???

Each type implements draw() in its own way — no inheritance needed.

---

# Operator Overloading

```rust
use std::ops::Add;

#[derive(Debug)]
struct Point { x: i32, y: i32 }

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point { x: self.x + other.x, y: self.y + other.y }
    }
}
```

???

Implement Add, Sub, Mul, Div, Rem, PartialEq, PartialOrd, Index, Not, or other special traits to do operator overloading

---

# Operator Overloading

```rust
fn main() {
    let p1 = Point { x: 1, y: 1 };
    let p2 = Point { x: 2, y: 2 };
    
    println!("{:?}", p1 + p2);
}
```

---

# Generics

```rust
fn id<T>(e: T) -> T {
    e
}
```

---

# Into and From

```rust
pub trait From<T> {
    fn from(value: T) -> Self;
}

pub trait Into<T> {
    fn into(self) -> T;
}
```

---

# Into and From

```rust
struct Point {
    x: i32,
    y: i32,
}

impl From<(i32, i32)> for Point {
    fn from(pair: (i32, i32)) -> Self {
        Point { x: pair.0, y: pair.1 }
    }
}
```

???

- Into<U> for T -> Automatically available if From<T> for U exist

---

# Into and From

```rust
fn print_point<P: Into<Point>>(p: P) {
    let point: Point = p.into();
    println!("{:?}", point);
}
fn main() {
    print_point((1, 2));
}
```

---

# Display and Debug

```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 10, y: 20 };
*    println!("{:?}", p);        // prints: Point { x: 10, y: 20 }
*    println!("{:#?}", p);       // pretty-printed multi-line:
                                 // Point {
                                 //     x: 10,
                                 //     y: 20,
                                 // }
}
```

---

# Display and Debug

```rust
use std::fmt;

// Implement Display manually
impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}

fn main() {
    let p = Point { x: 10, y: 20 };

*    println!("{}", p);   // Output: (10, 20)
}
```

---

# Blanket implementations

```rust
// Blanket impl for all types
 impl`<T>` From<T> for `T` {
    fn from(t: T) -> T {
        t
    }
}
```

--

```rust
// Blanket impl for Display types (using Trait Bounds)
impl`<T: Display>` ToString for `T` {
    fn to_string(&self) -> String {
        format!("{}", self)
    }
}
```

---

# Generics and Trait Bounds

```rust
fn render<T: Drawable>(item: &T) {
    item.draw();
}

fn main() {
    let c = Circle { radius: 5.0 };
    let r = Rectangle { width: 3.0, height: 4.0 };

    // pass any Drawable type to render
    render(&c);
    render(&r);
}
```

[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2024&gist=f9f1ba3f36b95f55701493e7c4794f61)

???

- Traits constraining generic types

---

# Generics and Trait Bounds

```rust
fn render<T: Drawable>(item: &T) {
    item.draw();
}

fn main() {
    let s = String::from("Hello");
    render(&s);
}
```

???

- Try to pass a String that doesn't implement Drawable

---

# Generics and Trait Bounds

```rust
error[E0277]: the trait bound String: Drawable is not satisfied
  --> src/main.rs:32:12
   |
32 |     render(&s);
   |     ------ ^^ the trait Drawable is not implemented for String
   |     |
   |     required by a bound introduced by this call
   |
   = help: the following other types implement trait Drawable:
             Circle
             Rectangle
note: required by a bound in render
  --> src/main.rs:26:14
   |
26 | fn render<T: Drawable>(item: &T) {
   |              ^^^^^^^^ required by this bound in render
```

---

# impl Traits (in argument position)

```rust
fn render<`T: Drawable`>(item1: &`T`, item2: &`T`) {
    item1.draw();
    item2.draw();
}
```

or

```rust
fn render(item1: &`impl Drawable`, item2: &`impl Drawable`) {
    item1.draw();
    item2.draw();
}
```

???

- APITs vs Generics with Trait Bounds
- Both accept types that implement Drawable
- Static dispatch and checked at compile time
- Monomorphization
- Both provide parametric polymorphism or universal type (∀T)
- The caller chooses the type
- Almost identical, difference multiple args

---

# impl Traits (in return position)

```rust
fn make_drawable<T: Drawable>() -> T {
    Circle { radius: 5.0 }
}
```

or

```rust
fn make_drawable() -> impl Drawable {
    Circle { radius: 5.0 }
}
```

???

- RPITs vs Generics with Trait Bounds
- Both return something that implements Drawable
- Static dispatch and checked at compile time
- Monomorphization
- Return some type T that impls Drawable Both provide parametric polymorphism or existential type (∃T)
- The function chooses the type
- Represented by an opaque type internally
- Almost identical, difference multiple args

---

# Generics and Trait Bounds vs impl Trait

```rust
fn make_drawable() -> impl Drawable {
    Circle { radius: 5.0 }
}

fn main() {
    let d = make_drawable();
}
```

[Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2024&gist=64e19ca75d6f1f99ccc4a71ee88cf481)

---

# Generics and Trait Bounds vs impl Trait

```rust
fn make_drawable<T>() -> T where T: Drawable {
    Circle { radius: 5.0 }
}

fn main() {
    let d = make_drawable();
}
```

---

# Generics and Trait Bounds vs impl Trait

```rust
error[E0308]: mismatched types
  --> src/main.rs:27:5
   |
26 | fn make_drawable<T>() -> T where T: Drawable {
   |                  -       -
   |                  |       |
   |                  |       expected `T` because of return type
   |                  |       help: consider using an impl return type: `impl Drawable`
   |                  expected this type parameter
27 |     Circle { radius: 5.0 }
   |     ^^^^^^^^^^^^^^^^^^^^^^ expected type parameter `T`, found `Circle`
   |
   = note: expected type parameter `T`
                      found struct `Circle`
   = note: the caller chooses a type for `T` which can be different from `Circle`
```

---

# Generics and Trait Bounds vs impl Trait

```rust
fn make_drawable<T>() -> T where T: Drawable + Default {
    T::default()
}

fn main() {
    let d: Circle = make_drawable();
    // or
    // let d = make_drawable::<Circle>();
}
```

???

- Where clauses
- Trait bounds chaining +
- turbofish operator

---

# Type Alias Impl Trait


```rust
type DrawableImpl = impl Drawable;

fn make_drawable() -> DrawableImpl {
    Circle { radius: 5.0 }
}
```

???

- TAIT

---

# Type Alias Impl Trait

```rust
type DrawableImpl = impl Drawable;

fn make_circle() -> DrawableImpl {
    Circle { radius: 5.0 }
}

fn make_rectangle() -> DrawableImpl {
    Rectangle { width: 3.0, height: 4.0 } // Error! not the same concrete type
}
```

???

- Each alias corresponds to one opaque concrete type
- Defining uses of opaque types

---

# Dynamic Dispatch: Trait Objects

- What if we want to store multiple types that implement the same trait?
- dyn traits

---

# Dynamic Dispatch: Trait Objects

```rust
fn draw_all(drawable: Vec<&dyn Drawable>) {
    for d in drawable {
        println!("{:?}", d.draw());
    }
}

fn main() {
    let drawable = vec![
       &Circle { radius: 5.0 } as &dyn Drawable,
       &Rectangle { width: 3.0, height: 4.0 } as &dyn Drawable,
    ];

    draw_all(drawable);
}
```

???

dyn Trait = dynamic dispatch, heap-allocated, runtime polymorphism.
impl Trait = static dispatch, monomorphized at compile time.

---

# Associated types

```rust
pub trait Iterator {
    type Item; // Associated type

    fn next(&mut self) -> Option<Self::Item>;

    // many useful default methods (map, filter, collect, etc.)
}
```

???

---

# Writing your own Iterator

```rust
struct Counter {
    current: u32,
    end: u32,
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        if self.current < self.end {
            let result = self.current;
            self.current += 1;
            Some(result)
        } else {
            None
        }
    }
}
```

---

# Iterators

```rust
fn main() {
    let result: Vec<_> = Counter { current: 1, end: 10 }
        .filter(|x| x % 2 == 0)
        .map(|x| x * x)
        .collect();
    
    println!("{:?}", result); // [4, 16, 36, 64, 100]
}
```

???



---

# IntoIterator Trait

```rust
pub trait IntoIterator {
    type Item;
    type IntoIter: Iterator<Item = Self::Item>;

    fn into_iter(self) -> Self::IntoIter;
}
```

???

---

# IntoIterator Trait

```rust
for x in v {
    ...
}
```

expands to

```rust
for x in v.into_iter() { ... }
```

???

- Convert into an iterator
- That’s why for works for arrays, slices, Vecs, HashMaps, etc. — each implements the IntoIterator trait:

---

# Auto traits and Marker traits

- Marker traits are traits that don’t define any methods, they only mark types with a property.
- Auto traits are special traits in Rust that are automatically implemented by the compiler for types that satisfy certain conditions.
- They’re mostly used to track properties of a type (like thread-safety or ownership semantics), not behavior.
- Auto traits are marker traits
- Send, Sync, Unpin, Sized, etc

---

# Auto traits

```rust
use std::thread;

fn main() {
    let s = String::from("hello");

    thread::spawn(move || {
        println!("{}", s);
    })
    .join()
    .unwrap();
}
```

???

- Works fine, because String is Send — it can safely be moved to another thread.

---

# ?Sized

```rust
fn print_debug<T: Debug + ?Sized>(x: &T) {
    println!("{:?}", x);
}

fn main() {
    let s: &str = "hello";        // str is unsized
    let arr: &[i32] = &[1, 2, 3]; // [i32] is unsized

    print_debug(s);
    print_debug(arr);
}
```

---

# Function Traits (Fn, FnMut, FnOnce)

- Rust treats functions and closures as types that implement one of three traits:
- FnOnce takes self by value, can consume captured values
- FnMut: takes self by mutable reference, can capture environment mutable
- Fn: takes self by reference, can capture environment immutable
- Fn: FnMut: FnOnce

---

# FnOnce

```rust
let s = String::from("hello");

let consume = || {
    println!("{}", s);
    // s is moved here
};

// consume() consumes s
consume();
// consume(); // cannot call again, s is gone
```

---

# FnMut

```rust
let mut count = 0;

let mut add_one = || {
    count += 1;
    println!("{}", count);
};

add_one(); // 1
add_one(); // 2
```

---

# FnOnce

```rust
let greeting = "hello";

let say_hi = || println!("{}", greeting);

say_hi(); // hello
say_hi(); // hello
```

---

# Coherence

- In Rust, coherence ensures that for any given type and trait, there is exactly one implementation that the compiler can find.
- For every trait T and type A, there must be at most one impl T for A that’s visible to any given crate.
- The orphan rule is the main mechanism enforcing coherence.

---

# Coherence (Orphan Rule)

You can implement a trait for a type only if either:
- The trait is defined in your crate, or
- The type is defined in your crate.

---

# Dyn compatibility (Trait object safety)

A trait is object safe if all of its methods can be called on a trait object.

- All supertraits must also be dyn compatible.
- Sized must not be a supertrait. In other words, it must not require Self: Sized. 
- It must not have any associated constants.
- It must not have any associated types with generics.
- All associated functions must either be dispatchable from a trait object or be explicitly non-dispatchable

---

# Higher Ranked Trait Bounds (HRTBs)

```rust
fn apply_to_str<F>(f: F)
where
    F: for<'a> Fn(&'a str) -> usize, // no HRTB
{
    let s1 = "hello";
    let s2 = String::from("world!");
    println!("{}", f(&s1));
    println!("{}", f(&s2));
}

fn main() {
    let f = |s: &str| s.len();
    apply_to_str(f);
}
```

---

# Recap

- Polymorphism
- Trait objects and dynamic dispatch dyn Traits
- Trait bounds
- Static dispatch (Monomorphization)
- Default methods
- Auto traits (Send, Sync, Unpin, Sized, etc)
- Marker traits (Copy, Fn, FnMut, FnOnce, etc)
- Derive macros

---

# Recap

- Trait inheritance (supertraits) Eq: PartialEq
- Blanket implementations
- Associated types
- APITs, RPITs, TAITs
- Generic associated types (GATs)
- Trait coherence / orphan rule
- Specialization (unstable)
- Object Safety
- HRTBs (Higher-Ranked Trait Bounds)

???

https://quinedot.github.io/rust-learning/index.html

---

class: center
name: title
count: false

<img src="content/images/rust-logo-blk.svg" alt="Rust logo" width="250rem" height="auto">

# Thanks

.grey[Github/Everywhere: spastorino]<br/>
.grey[Email: spastorino@gmail.com]
