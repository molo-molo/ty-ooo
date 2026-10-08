# Out-of-order references

## Disabled by default

Without `allow-out-of-order-references`, a load whose first binding is only reachable later is
reported, mirroring Python's runtime behavior.

```py
a = 1.0 + b  # error: [unresolved-reference]
b = 2.0
```

## Forward references enabled

A name that is never bound anywhere is still reported.

```toml
[analysis]
allow-out-of-order-references = true
```

```py
a = 1.0 + b
b = 2.0
```

```py
a = 1.0 + b
b = 2.0
c = a + b + d  # error: [unresolved-reference]
```

## Types are taken from all reachable bindings

```toml
[analysis]
allow-out-of-order-references = true
```

```py
import random

x = y
if random.random() > 0.5:
    y = 1

reveal_type(x)  # revealed: Literal[1]
```

## Narrowing is unaffected

```toml
[analysis]
allow-out-of-order-references = true
```

```py
def f(x: int | None):
    if x is not None:
        reveal_type(x)  # revealed: int
    else:
        reveal_type(x)  # revealed: None
```

## Class body

```toml
[analysis]
allow-out-of-order-references = true
```

```py
class C:
    y = b
    b = 1

reveal_type(C.y)  # revealed: int
```

## Member loads

Only the symbol that owns the place expression is retried, so the receiver of a member load is
resolved exactly as it is without the setting.

```toml
[analysis]
allow-out-of-order-references = true
```

```py
obj = 1
obj.x  # error: [unresolved-attribute]
```

## Enclosing scope of a lazy scope

A free variable read in a function body already resolves definitions from the whole enclosing
scope, so the read itself does not depend on the setting.

```toml
[analysis]
allow-out-of-order-references = true
```

```py
def f():
    return later

later = 1

reveal_type(later)  # revealed: Literal[1]
```
