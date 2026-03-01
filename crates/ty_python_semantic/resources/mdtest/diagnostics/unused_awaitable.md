# Unused awaitable

## Basic coroutine not awaited

Calling an `async def` function produces a coroutine that must be awaited.

```py
async def fetch() -> int:
    return 42

async def main():
    fetch()  # error: [unused-awaitable]
```

## Awaited coroutine is fine

```py
async def fetch() -> int:
    return 42

async def main():
    await fetch()
```

## Assigned coroutine is fine

```py
async def fetch() -> int:
    return 42

async def main():
    coro = fetch()
    await coro
```

## Coroutine passed to a function

When a coroutine is passed as an argument rather than used as an expression statement, no diagnostic
should be emitted.

```py
async def fetch() -> int:
    return 42

async def main():
    print(fetch())
```

## Top-level coroutine call

The lint fires even outside of `async def`, since the coroutine is still discarded.

```py
async def fetch() -> int:
    return 42

fetch()  # error: [unused-awaitable]
```

## Union of awaitables

When every element of a union is awaitable, the lint should fire.

```py
from types import CoroutineType
from typing import Any

def get_coroutine(flag: bool) -> CoroutineType[Any, Any, int] | CoroutineType[Any, Any, str]:
    raise NotImplementedError

async def main():
    get_coroutine(True)  # error: [unused-awaitable]
```

## Union with non-awaitable

When a union contains a non-awaitable element, the lint should not fire.

```py
from types import CoroutineType
from typing import Any

def get_maybe_coroutine(flag: bool) -> CoroutineType[Any, Any, int] | int:
    raise NotImplementedError

async def main():
    get_maybe_coroutine(True)
```

## Non-awaitable expression statement

Regular non-awaitable expression statements should not trigger this lint.

```py
def compute() -> int:
    return 42

def main():
    compute()
```

## Dynamic type

`Any` and `Unknown` types should not trigger the lint.

```py
from typing import Any

def get_any() -> Any:
    return None

async def main():
    get_any()
```
