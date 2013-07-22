Understanding closures in Rust (DRAFT)
======================================

In this post, I'm going to look at closures in Rust: what they are, how they play with the type system, the memory model, and how they compare to lambda expressions in C++11 and blocks in Objective-C. For those of you who don't know, [Rust][1] is an open source and very ambitous systems programming language, currently under active development by Mozilla and contributors.

Before we begin, I need to mention that I'm a beginner in Rust. I haven't found much documentation about closures, so I decided to figure them out by myself (with great help from #rust on IRC) and share my findings with others.

Another warning: I'm not covering common use cases nor best practices related to closures here. We will only look at how they work, how to store them and call them in our own code.

## Prerequisites

In order for you to keep up with the explanations below, you'll need to have a basic understanding of Rust: its syntax, semantics of its three pointer types ([tutorial 1][2], [tutorial 2][3], [blog post][4]) and [memory management][5], type inference, etc. The [tutorial][6] and [Rust for Rubyists book][7] (no Ruby knowledge required) provide a good intro to the language in general.

The tutorial provides a brief [introduction][8] to closures, but to really understand how they work we'll need to go deeper and look closely at how closures and memory management interact with each other in Rust.

## Beware

Rust is under active development at the moment, things may and will change. Even the tutorial mentions that certain aspects of how closures work will change over time. I'm using Rust 0.7 in this post. Every code sample is followed by the output of `rust run <corresponding source file>`.

## What is a closure?

A closure is an anonymous function that can capture variables from the surrounding scope. The value of any variable that is defined outside of the closure body but is referenced within it will be captured and passed around along with the closure itself.

We can think of a closure in Rust as a pair `(F, E)` where `F` is a function pointer to the anonymous function generated from the closure's body and `E` is the environment captured by the closure. We'll take a closer look at the environment part in the next section. For now, let's look at some examples.

In the first example, we'll create a closure of one argument that captures an integer value and returns a sum of this value and the argument:

```
fn main() {
    let a = 5;
    let adder = |x: int| x + a;

    // We can now invoke adder as an ordinary function
    println(fmt!("adder(1) = %d", adder(1)));
}

---
adder(1) = 6
```

A closure is written as a list of arguments between pipes (`|`), followed by an optional return type and a single expression. In the sample above, the compiler is able to infer the return type. Alternatively, we could write all the types ourselves:

```
let adder = |x: int| -> int x + a;

// or
let adder: &fn(int) -> int = |x| x + a;

// or even
let adder: &fn(int) -> int = |x: int| -> int x + a;
```

In the second example, our closure captures a mutable value:

```
fn main() {
    let mut a: int = 0;
    let adder = |x: int| { a += 1; x + a };

    a = 10;

    println(fmt!("adder(1) = %d", adder(1)));
    println(fmt!("a = %d", a));
}

---
01b-local-mut-closure.rs:5:4: 5:5 warning: value assigned to `a` is never read [-W dead-assignment (default)]
01b-local-mut-closure.rs:5     a = 10;
                               ^
adder(1) = 12
a = 11
```

The warning is actually a bug ([issue 7966][9]), so disregard it. What is important to note here is that the call `adder(1)` returns `12`. That is, when `adder` is invoked, it gets the latest value of `a` which is `10`, adds `1` to it and adds that to the argument `x`. In the end, `11` is stored in `a`, and thus the closure has affected its surrounding scope.

## Understanding closures

A closure value (the actual thing that lives in the memory during the program execution) is comprised of two components: function pointer `F` and environment `E`. The function pointer has `static` lifetime, meaning that it lives throughout the whole duration of the program execution. This makes sense, doesn't it: the code is compiled into the final binary and that binary is loaded into memory at program startup. So the function code will be residing in the virtual memory at all times during the program execution and its address will never change.

With ordinary function pointers, that is all you need to know. You can take the address of a function, pass it around as a borrowed pointer and feel perfectly safe about doing this. For example,

```
fn add(x: int) -> int {
    x + 1
}

fn sub(x: int) -> int {
    x - 1
}

fn doer(a: int, f: &fn(int) -> int) -> int {
    f(a)
}

fn main() {
    let a = 10;
    println(fmt!("%d", doer(a, add)));
    println(fmt!("%d", doer(a, sub)));
}

---
11
9
```

Nothing surprising here.

The `E` component of a closure – its environment – is what makes closures trickier than mere function pointers. In order to pass closures around we need to specify their environments' lifetime. There are three possiblilities: environment on the stack, environment in an owned box, and environment in a managed box.

When talking about the lifetime of a closure, what we really mean is the lifetime of its environment (remember, the function pointer always has `static` lifetime). This notion is reflected in how different types of closures are declared:

```
// environment on the stack (the default when type is omitted)
let f1: &fn(int) -> int = |x| x + 1;

// environment in an owned box
let f2: ~fn(int) -> int = |x| x + 1;

// environment in a managed box
let f3: @fn(int) -> int = |x| x + 1;
```

The semantics of each of the three types of closures are slightly different. Only the latter two are true first-class values while the first one can only be passed to other functions, not stored or returned. Let's take a look at each one in turn.

But before we do that, let me mention a trivial case of a closure – one that does not capture any variables:

```
fn get_adder() -> &fn(int) -> int {
    |x| x + 1
}

fn main() {
    let adder = get_adder();
    println(fmt!("%d", adder(4)));
}

---
5
```

Since the closure body does not reference any variables from the outer scope, it is equivalent to an ordinary function. In this case, "passing the closure" means "passing the function pointer" as such closures don't have any more restrictions that ordinary function pointers. We could use either closure type in the code above with the same visible effect.

Now let's look at a more interesting case of closures that do capture their environments.

## Borrowed closures

Take a look at this code sample:

```
fn get_adder(a: int) -> &fn(int) -> int {
    |x| x + a
}

fn main() {
    let adder = get_adder();
    println(fmt!("%d", adder(4)));
}

---
03-borrowed-closure.rs:1:40: 2:13 error: cannot infer an appropriate lifetime due to conflicting requirements
03-borrowed-closure.rs:1 fn get_adder(a: int) -> &fn(int) -> int {
03-borrowed-closure.rs:2     |x| x + a
03-borrowed-closure.rs:1:40: 3:1 note: first, the lifetime cannot outlive the block at 1:40...
03-borrowed-closure.rs:1 fn get_adder(a: int) -> &fn(int) -> int {
03-borrowed-closure.rs:2     |x| x + a
03-borrowed-closure.rs:3 }
03-borrowed-closure.rs:2:12: 2:13 note: ...due to the following expression
03-borrowed-closure.rs:2     |x| x + a
                                     ^
03-borrowed-closure.rs:1:40: 3:1 note: but, the lifetime must be valid for the anonymous lifetime #1 defined on the block at 1:40...
03-borrowed-closure.rs:1 fn get_adder(a: int) -> &fn(int) -> int {
03-borrowed-closure.rs:2     |x| x + a
03-borrowed-closure.rs:3 }
03-borrowed-closure.rs:1:40: 2:13 note: ...due to the following expression
03-borrowed-closure.rs:1 fn get_adder(a: int) -> &fn(int) -> int {
03-borrowed-closure.rs:2     |x| x + a
error: aborting due to previous error
```

Oops, that didn't work. But thanks to the verbose output, we can actually understand why it failed. By following each `error` and `note` explanation, we see that because the lifetime of `a` is limited to `get_adder`'s body, we cannot return a closure with borrowed environment because the environment will immediately become invalid.

But that's not the real underlying issue here. Even if we tried capturing the address of a variable with a longer lifetime, the closure's own environment would still be valid only within `get_adder`'s body. That's because the environment of a borrowed closure is always allocated on the stack. This effectively means that such a closure cannot be returned from a function – its environment would be immediately destroyed, and the borrowed pointer to it would become invalid. The only exception to this is the previous example of `|x: int| x + 1` which is a closure without an environment (basically just a function pointer).

So we can't return a borrowed closure and we can't store it in any other structure that outlives the stack frame the closure was defined in. What can we use those closures for?

As the Rust [tutorial][8] mentions, borrowed closures are a perfect fit for passing as arguments to a function. Remember our function pointer example:

```
fn add(x: int) -> int {
    x + 1
}

fn sub(x: int) -> int {
    x - 1
}

fn doer(a: int, f: &fn(int) -> int) -> int {
    f(a)
}

fn main() {
    let a = 10;
    println(fmt!("%d", doer(a, add)));
    println(fmt!("%d", doer(a, sub)));
}
```

We could turn both `add` and `sub` into borrowed closures only if we could guarantee they would outlive any scope they may be used in. It turns out, we can fairly easily provide such a guarantee:

```
fn complex_doer(a: int, i: int) {
    let add = |x: int| x + i;
    let sub = |x: int| x - i;

    println(fmt!("%d", doer(a, add)));
    println(fmt!("%d", doer(a, sub)));
}

fn doer(a: int, f: &fn(int) -> int) -> int {
    f(a)
}

fn main() {
    let a = 10;
    complex_doer(a, 1);
    complex_doer(a, 2);
}

---
11
9
12
8
```

In the code above, we define two closures that capture `i` and then pass them to `doer`. It is easy to see that both closures outlive the calls to `doer` because `doer` doesn't store the function pointer it is passed, it merely invokes it. Therefore it is legal to use borrowed closure here, it is also the cheapest type of closures for this task: if we used `~fn` or `@fn`, we'd have to pay with a heap allocation.

And the fact that we're using closures here allows us to customize `i` without writing multiple functions: because `doer` expects a one-argument function, there would be no way to customize the incremenet value other than by creating a separate function for each one. This pattern is useful for iteration and other stuff: we can construct a closure that references any values in its surrounding scope and pass it to another function that takes a function pointer and has no additional information about the closure's captured values or even of the fact that it is, in fact, a closure.

Let's finish this section off with an example of capturing a mutable variable:

```
fn complex_doer(a: int, i: int) {
    let mut mi = i;

    let add = |x: int| { let ret = x + mi; mi += 1; ret };
    let sub = |x: int| { let ret = x - mi; mi += 1; ret };

    println(fmt!("add: %d", doer(a, add)));
    println(fmt!("sub: %d", doer(a, sub)));

    println(fmt!("mi = %d", mi));
}

fn doer(a: int, f: &fn(int) -> int) -> int {
    f(a)
}

fn main() {
    let a = 10;
    complex_doer(a, 1);
}

--

add: 11
sub: 8
mi = 3
```

Here you can see how the two closures are able to mutate `mi` – a variable from the outer scope. And similarly to the immutable case, since both `mi` and the closures have the same lifetime, it is safe to use borrowed closures here.

## Boxed closures

Next, let's look at the types of closures that are truly first-class: managed and owned closures.

```
fn get_adder(a: int) -> @fn(int) -> int {
    |x| x + a
}

fn main() {
    let add10 = get_adder(10);
    println(fmt!("add10(1) = %d", add10(1)));
    println(fmt!("add10(2) = %d", add10(2)));

    let add3 = get_adder(3);
    println(fmt!("add3(1) = %d", add3(1)));
    println(fmt!("add3(2) = %d", add3(2)));
}

---
add10(1) = 11
add10(2) = 12
add3(1) = 4
add3(2) = 5
```

The `get_adder` function returns a closure with managed environment – it can be passed around freely and all of the captured variables are going to be stored on the heap.

We could also return an owned closure with the same visible effect:

```
fn get_adder(a: int) -> ~fn(int) -> int {
    |x| x + a
}
```

Here, all the rules defined for owned boxes are applied to owned closures as well – each one can have a single owner at any time. This actually means that the closure's _environment_ has a single owner. One notable consequence of this is that we can pass those closures between tasks:

```
fn get_adder(a: int) -> ~fn(int) -> int {
    |x| x + a
}

fn main() {
    let add1 = get_adder(1);
    do spawn {
        println(fmt!("add1(1) = %d", add1(1)));
        println(fmt!("add1(2) = %d", add1(2)));
    }
}

---
add1(1) = 2
add1(2) = 3
```

Trying to use `add1` after the `do spawn { ... }` block would result in the following error:

```
error: use of moved value: `add1`
```

Also, if we tried to use a managed closure, the compiler would complain about the impossibility of sending such a closure to another task:

```
fn get_adder(a: int) -> @fn(int) -> int {
    |x| x + a
}

fn main() {
    let add1 = get_adder(1);
    do spawn {
        println(fmt!("add1(1) = %d", add1(1)));
        println(fmt!("add1(2) = %d", add1(2)));
    }
    add1(3);
}

---
05a-task-closure.rs:8:37: 8:41 error: cannot capture variable of type `@fn:'static(int) -> int`, which does not fulfill `Send`, in a bounded closure
05a-task-closure.rs:8         println(fmt!("add1(1) = %d", add1(1)));
                                                           ^~~~
note: in expansion of fmt!
05a-task-closure.rs:8:16: 8:46 note: expansion site
05a-task-closure.rs:8:37: 8:41 note: this closure's environment must satisfy `Send`
05a-task-closure.rs:8         println(fmt!("add1(1) = %d", add1(1)));
                                                           ^~~~
note: in expansion of fmt!
05a-task-closure.rs:8:16: 8:46 note: expansion site
error: aborting due to previous error
```

That is by far the most significant difference between owned and managed closures: the former ones can have only one owner (like any other owned value). Apart from that, both closures allocate their environment on the heap and both cannot capture mutable values.

Did the last statement surprise you? Let's take a look at this simple example:

```
fn get_adder_mut() -> ~fn(int) -> int {
    let mut a = 5;
    |x| x + a
}

fn main() {
    let adder = get_adder_mut();
    println(fmt!("%d", adder(4)));
}

---
06-mut-closures.rs:3:12: 3:13 error: mutable variables cannot be implicitly captured
06-mut-closures.rs:3     |x| x + a
                                 ^
error: aborting due to previous error
```

Using a managed closure would result in the same error. Why is that?

If you think about it for a bit, it starts to make sense. A mutable variable cannot outlive the scope it was declared in. Consider this code:

```
fn f(mut b: int) {
    b = 10;
    println(fmt!("%d", b));
}

fn main() {
    let mut a = 5;
    f(a);
    a += 1;
    println(fmt!("%d", a));
}

---
10
6
```

Neither `f` can affect `a` in `main`, nor `main` can affect `b` in `f`. Both variables live in their respective stack frames and both are mutable in this sense that their owning scope can change their value in that scope. When we're trying to capture a mutable variable, we're most likely thinking that we'll be able to change that variable's value inside the closure. But that is not the case, and the compiler helps us out here by pointing out the source of a potential mistake.

So how do we capture a mutable value inside a closure? It was easy to do with borrowed closures, surely there has to be a way for boxed closures to do the same. And there is such a way – all we need to do is declare the captured value with appropriate storage type:

```
fn adder(a: int) -> @fn(int) -> int {
    let b = @mut a;
    |x| { let ret = x + *b; *b += 1; ret }
}

fn main() {
    let add1 = adder(1);
    println(fmt!("%d", add1(1)));
    println(fmt!("%d", add1(2)));
}

---
2
4
```

Here, we capture `b` which is a managed mutable box that initially contains a _copy_ of `a`. Each time the closure is invoked, we increment the value stored in `b`, hence the second call to `add1` returning `4`.

Unfortunately, capturing a mutable value is a bit more involved in the case of an owned closure. This might be a temporary issue, but for now, the workaround is as follows:

```
use std::cell::Cell;

fn adder(a: int) -> ~fn(int) -> int {
    let cell = ~Cell::new(a);
    |x| {
        let b = cell.take();
        let ret = x + b;
        cell.put_back(b+1);
        ret
    }
}

fn main() {
    let add1 = adder(1);
    println(fmt!("%d", add1(1)));
    println(fmt!("%d", add1(2)));
}

---
2
4
```

The difference in the implementation follows from the fact that mutability of an owned value is determined by the owner (the owning variable), not the value itself (as was the case with `@mut`). See the [tutorial][10] for an explanation of this.

And because we cannot capture a mutable variable inside a boxed closure, we need to use a special `Cell` container here that provides access to a single mutable location. Moreover, an instance of `Cell` itself is not owned, so we explicitly put it into an owned box so that it can't be used after the closure definition. See the docs for [more about `Cell`][11].

## Summing up

Let's review everything we've discovered about closures:

1. Closures allow us to define anonymous functions that capture values from their surrounding scope.

2. A closure is comprised of a function pointer `F` and a captured environment `E`. We can specify the storage of `E` by declaring the type of our closure as `&fn`, `~fn`, or `@fn`.

3. By default, closures are inferred to have type `&fn`. This kind of closures has its environment allocated in the current stack frame so it can't be returned from a function or stored in a value that outlives the current stack frame unless its environment is empty. Other than that, borrowed closures are more efficient than their boxed counterparts because they don't cause a heap allocation.

4. Closures with both owned (`~fn`) and managed (`@fn`) environments behave similarly and differ in the same ways as owned and managed pointers of any other type. One notable difference from the `&fn` closures is that the latter can capture mutable local variables (provided they have the same lifetime as the closure itself) whereas neither `~fn` nor `@fn` can do that (they can capture mutable values though).

5. Owned (`~fn`) and managed (`@fn`) closures are useful for the cases when a function needs to return a closure or store it in some data structure. The choice between `~` and `@` is made according to the general guidelines for using those kinds of pointers.

6. A function taking an argument of type `&fn(...) -> ...` can take any kind of closure or a mere function pointer:

        fn do_it(f: &fn()) {
            f()
        }

        fn some_func() {
            println("Some func");
        }

        fn main() {
            let f1 = || println("Borrowed closure");
            let f2: @fn() = || println("Managed closure");
            let f3: ~fn() = || println("Owned closure");

            do_it(f1);
            do_it(f2);
            do_it(f3);
            do_it(some_func);
        }

        ---
        Borrowed closure
        Managed closure
        Owned closure
        Some func

## Comparison with C++11 and Objective-C (WIP)

Closures in Rust fall somewhere between lambda expressions in C++ and blocks in Objective-C.

In C++, a lambda is just an object like any other. Passing it around will copy the whole environment from one stack frame to another. If you capture any variable from the lambda's surrounding scope by reference, it is your responsibility to make sure that the variable outlives the lambda. Otherwise, reading from an already dead variable inside the lamdba will result in undefined behavior.

In Objective-C, blocks always start on the stack, but move to the heap after first copy. Capturing by value is always safe, and capturing by reference can be done by specifying the `__block` storage type for individual variables like so:

```
typedef int (^BlockType)(void);

BlockType f() {
    __block int a = 0;
    return Block_copy(^{ a++; return a; });
}
```

To be able to return a block, we copy it to the heap. This means that `a` will be copied to the heap as well and it is safe to reference it inside the block. Subsequent calls to `Block_copy()` will only increase the reference count of the block if it is already on the heap.

In Rust we get neither behavior. We can't pass environments on the stack (there is no `fn` value, we can only get a pointer: `&fn`, `@fn`, or `~fn`), they need to be stored within an owned or a managed box. We can't promote a stack-based environment to the heap either. That is, a `&fn` closure cannot be returned from a function, because its environment lives in the current stack frame that is going to be destroyed after the current function returns. It can however be used for passing the closure as an argument so long as its own stack frame is alive. Returning a closure from a function always involves using `~fn` or `@fn` (except for the trivial case of empty environment).

---

That is all for now. Hopefully, I've given you a glimpse into the underlying mechanics of closures in Rust, and now you have a better understanding of how they work and you'll be able to start using them in your code.

For any questions regarding this post or any inaccuracies found in it, I'm available at: 1) my email; 2) @true_droid; 3) true_droid in #rust on IRC.

Thanks for reading.

  [1]: http://www.rust-lang.org/
  [2]: http://static.rust-lang.org/doc/tutorial.html#boxes
  [3]: http://static.rust-lang.org/doc/tutorial-borrowed-ptr.html
  [4]: http://pcwalton.github.io/blog/2013/03/18/an-overview-of-memory-management-in-rust/
  [5]: http://pcwalton.github.io/blog/2013/05/20/safe-manual-memory-management/
  [6]: http://static.rust-lang.org/doc/tutorial.html
  [7]: http://www.rustforrubyists.com/
  [8]: http://static.rust-lang.org/doc/tutorial.html#closures
  [9]: https://github.com/mozilla/rust/issues/7966
  [10]: http://static.rust-lang.org/doc/tutorial.html#ownership
  [11]: http://static.rust-lang.org/doc/std/cell.html

---
Tags: rust-lang, lambdas, proglang


Date: Mon Jul 22 23:15:54 EEST 2013
