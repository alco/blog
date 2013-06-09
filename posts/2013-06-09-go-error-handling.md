Error Handling in Go
====================

This is just a little observation of mine.

Programmers are lazy. I'm lazy too. If I want to hack something together
quickly, in Go I will write

```go
file, _ = os.Open(path)
// ...
// code that uses `file`
```

I'm explicitly ignoring the error returned as the second value from
`os.Open()`.

When this code is run and there is an error opening the file, the error will be
ignored and `file` will get the `nil` value. And you guessed it: the program will
blow up somewhere **after the offending line**. It's OK, file errors are
inevitable, so I should review my code and not ignore errors but deal with
them.

Now let's switch our mind into Erlang mode. In Erlang, there are runtime errors
similar to exceptions in other dynamic languages, but they are used to signal
programmer's errors. That is, if you get a runtime error, your code is broken
and you need to fix it before your program is run in production. Catching
those errors in code is **not** a good practice. Fairly similar to Go's `panic`.

For the kinds of errors that are expected to occur in production (file errors,
user input errors, etc.), Erlang has a convention of returning error values.
This is similar to Go, but with one important difference.

So programmers are lazy. I'm lazy too. If I want to hack something together
quickly, in Erlang I will write

```erlang
{ ok, File } = file:open(Path, [read]).
% ...
% code that uses `File`
```

In Erlang, identifiers that start with a lowercase letter are atoms -- constant
values that evaluate to themselves (`ok`, `read`). Variable names are
capitalized identifiers (`File`, `Path`).

Another peculiarity is that `=` is not an assigment operator, it's a match
operator. The expression on the left matches the one on the right when all
respective values in the expression on the left are equal to the ones on the
right. In the example above, we match a tuple with two elements to the return
value of `file:open()` call. Since our `File` variable is used for the first
time there, it will bind to whatever value sitting as the second element of the
returned tuple is. However, if `file:open()` returns anything other than a
tuple with two elements the first of which is the atom `ok`, the match will
fail and a runtime error will be raised.

Erlang's convention is to return `{ ok, Value }` in the case of success and `{
error, Reason }` in the case of an error. So when the code above is run and
there is an error opening the file, the program will blow up **immediately on
the offending line** with a runtime error and a stacktrace.

The difference to Go's behavior might seem small, but to me it makes all the
difference in the world.

If you'd like to know more about Erlang's error handling philosophy, google
"erlang let it crash".


Tags: proglang, go, erlang, error handling

Date: Sun Jun  9 12:50:55 EEST 2013
