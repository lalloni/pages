# Go recommendations

## Logging library

For any serious production ready system the std lib logging library lacks key features.

Using a library which implements leveled and structured logging is a must.

For performance reasons I recommend using [Uber'z zap](https://github.com/uber-go/zap) above other (good) alternatives that exist. *Please learn how to use it right.*

## Avoid using `new`

Generally speaking is better to avoid using the `new` keyword as it is seldom needed.

For allocating a struct type and getting a reference to it, please use consistently the syntax `a := &some.StructType{}`.
This syntax is very close to the syntax you would use for initializing the struct with some non-zero values, so you have less cognitive burden (only one syntax for both use cases).

## One-letter receiver names

Please use one-letter names for receiver variables.

## Prefer one-letter variable names

Most variables names should be one-lettered.

Exceptions are function parameters or return variables of primitive or repeated types and when we need to be explicit about semantics.

## Do not use `log.Fatal`

We should never use `log.Fatal(...)`, in fact it should be considered the same as using `panic(...)` because it will effectively render useless all proper shutdown code of your app.

Instead we should return an error to be properly handled by calling code.

