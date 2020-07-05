Custom Operators
================

{{#include ../links.md}}

For use as a DSL (Domain-Specific Languages), it is sometimes more convenient to augment Rhai with
customized operators performing specific logic.

`Engine::register_custom_operator` registers a keyword as a custom operator.


Example
-------

```rust
use rhai::{Engine, RegisterFn};

let mut engine = Engine::new();

// Register a custom operator called 'foo' and give it
// a precedence of 140 (i.e. between +|- and *|/)
engine.register_custom_operator("foo", 140).unwrap();

// Register the implementation of the customer operator as a function
engine.register_fn("foo", |x: i64, y: i64| (x * y) - (x + y));

// The custom operator can be used in expressions
let result = engine.eval_expression::<i64>("1 + 2 * 3 foo 4 - 5 / 6")?;
//                                                    ^ custom operator

// The above is equivalent to: 1 + ((2 * 3) foo 4) - (5 / 6)
result == 15;
```


Alternatives to a Custom Operator
--------------------------------

Custom operators are merely _syntactic sugar_.  They map directly to registered functions.

Therefore, the following are equivalent (assuming `foo` has been registered as a custom operator):

```rust
1 + 2 * 3 foo 4 - 5 / 6     // use custom operator

1 + foo(2 * 3, 4) - 5 / 6   // use function call
```

A script using custom operators can always be pre-processed, via a pre-processor application,
into a syntax that uses the corresponding function calls.

Using `Engine::register_custom_operator` merely enables a convenient short-cut.


Must Follow Variable Naming
--------------------------

All custom operators must be _identifiers_ that follow the same naming rules as [variables].

```rust
engine.register_custom_operator("foo", 20);     // 'foo' is a valid custom operator

engine.register_custom_operator("=>", 30);      // <- error: '=>' is not a valid custom operator
```


Binary Operators Only
---------------------

All custom operators must be _binary_ (i.e. they take two operands).
_Unary_ custom operators are not supported.

```rust
engine.register_custom_operator("foo", 140).unwrap();

engine.register_fn("foo", |x: i64| x * x);

engine.eval::<i64>("1 + 2 * 3 foo 4 - 5 / 6")?; // error: function 'foo (i64, i64)' not found
```


Operator Precedence
-------------------

All operators in Rhai has a _precedence_ indicating how tightly they bind.

The following _precedence table_ show the built-in precedence of standard Rhai operators:

| Category            |                                        Operators                                        | Precedence (0-255) |
| ------------------- | :-------------------------------------------------------------------------------------: | :----------------: |
| Assignments         | `=`, `+=`, `-=`, `*=`, `/=`, `~=`, `%=`,<br/>`<<=`, `>>=`, `&=`, <code>\|=</code>, `^=` |         0          |
| Logic and bit masks |                        <code>\|\|</code>,  <code>\|</code>, `^`                         |         30         |
| Logic and bit masks |                                        `&`, `&&`                                        |         60         |
| Comparisons         |                            `==`, `!=`, `>`, `>=`, `<`, `<=`                             |         90         |
|                     |                                          `in`                                           |        110         |
| Arithmetic          |                                        `+`, `-`                                         |        130         |
| Arithmetic          |                                      `*`, `/`, `~`                                      |        160         |
| Bit-shifts          |                                       `<<`, `>>`                                        |        190         |
| Arithmetic          |                                           `%`                                           |        210         |
| Object              |                                 `.` _(binds to right)_                                  |        240         |
| _Others_            |                                                                                         |         0          |

A higher precedence binds more tightly than a lower precedence, so `*` and `/` binds before `+` and `-` etc.

When registering a custom operator, the operator's precedence must also be provided.