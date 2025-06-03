---
title: Self-referential structs in Rust
description: Exploration of self-referential structs, Pin, Arenas and Ouroboros
slug: rust-self-referential-struct
published_date: 2025-05-31 16:01:48 +0000
layout: post.liquid
data: {
  nice_date: '31st May 2025, Paphos',
  avatar: 'https://gravatar.com/avatar/90ab307819ea66a7bbebe23cab35288b?size=256',
  avatarAlt: 'Federico Blancato',
  author: 'Federico Blancato',
  bio: 'I build things',
  contact: 'https://www.linkedin.com/in/federico-blancato-86139a84/',
}
---

<hgroup>

### {{ page.title }}

#### {{ page.data.nice_date }}

</hgroup>

&nbsp;

In Rust, there are a few paradigms that are more difficult to implement than in most other languages.
Today, we'll look at one of those: the self-referential structs.

A self-referential struct contains a field that borrows from another field of the same struct.

First of all, let's start with a simple example of why we might want to use this kind of pattern.

Let's imagine we want to write a csv parser, but we want to avoid extra memory allocation. One way to do that could be to return pointers
to the parsed fields, like so:

```rust
use std::io::{self, BufRead};

struct CsvRecord<'a> {
    line: String,
    fields: Vec<&'a str>,
}

fn load_record(line: String) -> CsvRecord<'_> {
    let mut record = CsvRecord {
        line,
        fields: Vec::new(),
    };

    record.fields = record.line.split(',').collect();
    record
}

fn main() -> io::Result<()> {
    let stdin = io::stdin();

    for line in stdin.lock().lines() {
        let record = load_record(line?);
        println!("{} {:?}", record.line, record.fields);
    }
    Ok(())
}
```

This example will actually not compile.
There are multiple issues with this code, we will explore them one by one, to dig deeper into self-referential data structures, but first, let's consider
that there are easier solutions to solve this in such a simple case, so let's get them out of the way:

- Store the owned data outside the struct and let the struct hold only references. If we don't put the data and the references to it in the same data structure, we could have the caller owned data live longer than the references to it.
- Store ranges rather than pointers. In this simple case, we could store the ranges of each one of the csv fields (ie: `fields: [(0..2), (4..5)]`) and avoid the references altogether

Those two straightforward approaches should be considered first, but this is not always possible.
So let's first explore what's so hard about these data structures, and that will bring us to other possible solutions that will be more flexible and accommodate for more complex real-life scenarios.

### Rust doesn't notify when a value's memory address changes

In Rust, the compiler is always allowed to move values to new memory addresses and doesn't notify that the address has changed.
Moving ownership might also change the memory address, like in this simple example:

```rust
struct Foo(String);

fn main() {
    let foo = Foo("foo".to_string());
    println!("ptr1 = {:p}", &foo);
    let bar = foo;
    println!("ptr2 = {:p}", &bar);
}

```

When you run this code, you will notice that the moving of `foo` into `bar`, will move the struct address, so the two printed addresses will be different.
Any pointers to the previous memory address will be pointing to an invalid address, but luckily safe Rust prevents this:
Moving the value will make any references to the old one invalid.

Heap allocations are stable between reassignment and function calls, and a simple move will not change the memory address. This will print the same address:

```rust
struct Foo(String);

fn main() {
    let foo = Foo("foo".to_string());
    println!("ptr1 = {:p}", foo.0.as_ptr());
    let bar = foo;
    println!("ptr2 = {:p}", bar.0.as_ptr());
}

```

So we could in theory have a stable pointer to the heap content, but still, this won't prevent safe functions from moving the address, like `mem::replace`, and that will break all the references pointing to it (and again, we need to use unsafe Rust to make this possible)

```rust
use std::ptr::NonNull;

struct Ref {
    data: String,
    ptr: NonNull<u8>,
}
fn main() {
    let data = String::from("foo");
    let mut boxed = Box::new(Ref {
        data,
        ptr: NonNull::dangling(),
    });
    boxed.ptr = NonNull::new(boxed.data.as_ptr() as *mut u8).unwrap();
    unsafe {
        println!(
            "second byte ptr  : {:p}  value: {}",
            boxed.ptr,
            *boxed.ptr.as_ptr()
        );
    }
    let _ = std::mem::replace(&mut boxed.data, String::from("foo"));
    unsafe {
        println!(
            "second byte ptr  : {:p}  value: {}",
            boxed.ptr,
            *boxed.ptr.as_ptr()
        );
    }
}
```

In the above example, we are replacing the `String` inside the box with another one.
The original string is dropped, and the pointer is then dangling and pointing to freed memory.
We had to use unsafe to access this dangling pointer, as we cannot trust the memory address to be stable,
and we want safeguards from the compiler, rather than relying on our ability to track and spot possible issues manually.

This can be achieved via pinning with `std::pin::Pin`.
`std::pin::Pin` is a wrapper in Rust's standard library that guarantees that the value inside the `Pin` will not be moved.
When a value is pinned, with some caveats that we're gonna discuss shortly, you won't be able to get a mutable reference to the wrapped value, making it impossible to change the underlying memory.

```rust
let mut boxed = Box::pin(Ref {
    data: "foo".to_owned(),
    ptr: NonNull::dangling(),
   _pin: PhantomPinned,
});
```

This will not only ensure that the compiler will prevent automatic moving of data like the ownership change, but also prevent user operations like `mem::replace`.

As you might have noticed, not only are we using `Box::pin` to wrap our struct, but we also added a new field containing a `PhantomPinned` marker.

This is due to an auto trait called `Unpin`. Auto traits are traits that are implemented for all the types unless explicitly opted out.
If a type implements `Unpin` you will be able to use operations like `mem::replace` on it even if the type is pinned.

The reason for having all the types be automatically `Unpin` is to alleviate the reduced ergonomics of APIs that require the use of `Pin` for soundness for some types, but which also want to be used by other types that don’t care about pinning.
This way from the user perspective, those types will behave as if they were not pinned in the first place.

That's not what we want for the example above, so we're opting out of `Unpin` by adding the `_pin: PhantomPinned` marker trait.
Every type with this marker trait will not implement `Unpin` by default.

### Unexpressible lifetimes

The other issue with self-referential data structures is that there is no way to express the lifetime of a reference tied to the lifetime of a struct.
So we need to use unsafe and manually ensure that references are still valid.
For example in this struct, this reference could have a longer lifetime than the data it references:

```rust
struct CsvRecord<'a> {
    line: String,
    fields: Vec<&'a str>,
}
```

A solution could be to have in the language a lifetime that represents the lifetime of the struct itself, which currently doesn't exist in Rust, but it could work like so:

```rust
struct CsvRecord {
    line: String,
    fields: Vec<&'self str>,
}
```

(not valid Rust syntax—just illustrative)

This way, we always know that the lifetime of the reference won't outlive `line`.
So if such a lifetime existed, and by pinning our data, self-referential data structures would be possible. For now, given this requires a whole lot of unsafe code and manual checks, let's explore some libraries that will do that for us and provide a safer alternative. We're gonna explore two possible solutions: Arenas & Ouroboros.

### Arenas

Arenas are used to allocate a bigger block of memory altogether, that can be used to contain smaller objects. Then we can deallocate the arena all at once.
This has a great benefit in that all the allocated objects will share the same lifetime, so it's a great fit for our use case.
One of the crates that offers this functionality is `bumpalo`

```rust
use bumpalo::{
    Bump,
    collections::{CollectIn, Vec as BVec},
};
use std::io::{self, BufRead};

struct CsvRecord<'a> {
    line: &'a str,
    fields: BVec<'a, &'a str>,
}

fn load_record<'a>(arena: &'a Bump, line: String) -> CsvRecord<'a> {
    let mut record = CsvRecord {
        line: arena.alloc(line),
        fields: BVec::new_in(arena),
    };

    record.fields = record.line.split(',').collect_in(arena);
    record
}

fn main() -> io::Result<()> {
    let stdin = io::stdin();
    let arena = Bump::new();

    for line in stdin.lock().lines() {
        let record = load_record(&arena, line?);
        println!("{} {:?}", record.line, record.fields);
    }
    Ok(())
}
```

This is replicating the initial example. Now the string is allocated into the arena with `arena.alloc` and all the references collected in the arena as well.
Given than both now are in the arena, there are no lifetime issues.

### Ouroboros

This is another interesting approach, that is much more specific to self-referential structs. It contains some macros that will provide some safer methods to interact with those structs, without having to use unsafe. Here is what the previous example would look like in ouroboros.

```rust
use ouroboros::self_referencing;
use std::io::{self, BufRead};

#[self_referencing]
struct CsvRecord {
    line: String,
    #[borrows(line)]
    #[covariant]
    fields: Vec<&'this str>,
}

fn load_record(line: String) -> CsvRecord {
    CsvRecordBuilder {
        line,
        fields_builder: |line: &String| line.split(',').collect(),
    }
    .build()
}

fn main() -> io::Result<()> {
    let stdin = io::stdin();

    for line in stdin.lock().lines() {
        let record = load_record(line?);
        println!("{} {:?}", record.borrow_line(), record.borrow_fields());
    }
    Ok(())
}
```

The attribute macro `self_referencing` will create a new struct called `<YourStructName>Builder` that will build the self-referential type. You must mark which fields are borrowing what, thanks to the attribute macro `borrows`. For those fields, you can now use a `'this` lifetime that is tied to the lifetime of the structure itself.

Then to create the actual self-referential type, you have to call structure builder, and for the borrowed fields you pass in a closure for `field_name_builder` that will take a reference to the borrowed data and return the borrowed values. Then you can build the struct (and the builder structs provide a bunch of other methods that you can check out in the [docs](https://docs.rs/ouroboros/latest/ouroboros/attr.self_referencing.html#mystructbuilder)).

One last remark is that `fields` are marked a `covariant` via an attribute macro.
Covariance means that we can use types with `'a` lifetime where `'a` is living at least as long as `'this`, as opposed to `not_covariant` where we could only use exact types with a `'this` lifetime. This is a property of the type itself, and in this specific example `Vec<T>` is covariant in `T` because it only hands out shared references to the elements.

This is necessary because it is not possible to determine the variance of a type inside macros. So we manually mark the type as `covariant` or `not_covariant` with the relative attribute macro. The macro will then generate or skip the `.borrow_*()` methods accordingly.

This will not impact the soundness of the code, as if you mark the type incorrectly, it will just not compile.

### Outro

In the end, true self-referential structs in Rust take a bit more work: either you park all your data in a shared arena so every slice stays valid, or you lean on a macro like ouroboros to pin and wire up those internal pointers for you.

Arenas give you one big, stable home for all your strings and their views, while ouroboros generates a builder that safely ties each field back to its owner.

With either pattern, you get efficient, zero-copy access without writing unsafe code yourself, just choose the approach that best fits your needs.
