# Clean Code Guideline for Python

## 1. Write code for humans first

Code is read much more often than it is written.
Optimize for **clarity**, not cleverness.

Bad:

```python
result = [x for x in items if x.a and not x.b and x.c > 10]
```

Better:

```python
result = [item for item in items if is_valid_item(item)]
```

## 2. Use meaningful names

Names should reveal intent.

Bad:

```python
d = {}
x = get()
```

Better:

```python
user_by_id = {}
current_order = get_order()
```

Rules:

* Use nouns for data: `user`, `invoice`, `retry_count`
* Use verbs for actions: `load_user`, `send_email`, `validate_input`
* Avoid vague names: `data`, `obj`, `tmp`, `stuff`, `manager`
* Avoid misleading abbreviations unless they are universally known

## 3. Functions must do one thing

A function should have **one responsibility** and do it well.

Bad:

```python
def create_user(request):
    validate_request(request)
    user = save_user_to_db(request)
    send_welcome_email(user)
    log_user_creation(user)
    return user
```

Better:

```python
def create_user(request: CreateUserRequest) -> User:
    validated_request = validate_user_request(request)
    user = save_user(validated_request)
    notify_user_created(user)
    return user
```

## 4. Keep functions small

Small functions are easier to understand, test, and change.

Good signs:

* fits on one screen comfortably
* has simple branching
* has one abstraction level

## 5. One level of abstraction per function

Do not mix high-level business logic with low-level details.

Bad:

```python
def process_order(order):
    if not order.items:
        raise ValueError("Order is empty")
    total = sum(item.price * item.quantity for item in order.items)
    with open("orders.log", "a") as f:
        f.write(f"Processed order {order.id}\n")
    return total
```

Better:

```python
def process_order(order: Order) -> Money:
    validate_order(order)
    total = calculate_order_total(order)
    log_order_processed(order)
    return total
```

## 6. Prefer explicit code over magical code

Python allows concise tricks, but concise is not always clear.

Avoid overly clever:

```python
value = config and config.get("timeout") or 30
```

Prefer:

```python
value = config.get("timeout", 30) if config else 30
```

## 7. Use descriptive comments sparingly

Comments should explain **why**, not **what**.
If code needs comments to explain what it does, improve the code.

Bad:

```python
# increment i
i += 1
```

Good:

```python
# Retry once because the upstream service occasionally returns stale data.
retry_count += 1
```

## 8. Avoid large argument lists

Too many arguments usually mean the function does too much.

Bad:

```python
def create_report(name, email, start_date, end_date, is_admin, locale, format):
    ...
```

Better:

```python
def create_report(request: ReportRequest) -> Report:
    ...
```

## 9. Use DTOs for input and output

Do not pass raw dictionaries, loosely structured payloads, or unclear tuples through the system.
Use explicit DTOs.

DTOs make code:

* easier to read
* easier to validate
* easier to type
* safer to refactor
* harder to misuse

Bad:

```python
def create_user(payload: dict) -> dict:
    ...
```

Better:

```python
from dataclasses import dataclass

@dataclass(slots=True)
class CreateUserRequest:
    email: str
    full_name: str
    is_active: bool

@dataclass(slots=True)
class UserResponse:
    id: int
    email: str
    full_name: str
    is_active: bool


def create_user(request: CreateUserRequest) -> UserResponse:
    ...
```

For application code:

* use `dataclass` for simple internal DTOs
* use Pydantic models where validation/parsing is needed
* use separate DTOs for input and output when their meanings differ

Do not mix:

* HTTP schemas
* domain entities
* ORM models
* transport payloads

Each layer should have its own clear models.

## 10. Type everything explicitly

Everything a function **gets** and **returns** should be explicitly typed.

That means:

* every argument has a type
* every return value has a type
* every public method has a type
* complex local values should be typed when it improves clarity

Bad:

```python
def get_discount(user, config):
    ...
```

Better:

```python
def get_discount(user: User, config: DiscountConfig) -> Percentage:
    ...
```

Bad:

```python
def parse(items):
    return [x.id for x in items]
```

Better:

```python
def parse(items: list[Item]) -> list[int]:
    return [item.id for item in items]
```

Bad:

```python
def find_user(user_id: int):
    ...
```

Better:

```python
def find_user(user_id: int) -> User | None:
    ...
```

Typing rules:

* never leave return type implicit in production code
* never use bare `dict`, `list`, `tuple` when a more specific type is possible
* prefer `UserDTO` over `dict[str, Any]`
* prefer `Sequence[str]` or `list[str]` over plain `list`
* prefer `Mapping[str, str]` over plain `dict`
* use `None` explicitly in return types when it is a real case
* avoid `Any` unless there is a very strong reason

Typing is not decoration.
Typing is part of the contract.

## 11. Prefer clear data models over primitive obsession

Do not pass primitives around when the value has business meaning.

Bad:

```python
def send_email(to: str, subject: str, body: str) -> None:
    ...
```

Better:

```python
@dataclass(slots=True)
class EmailMessage:
    to: str
    subject: str
    body: str


def send_email(message: EmailMessage) -> None:
    ...
```

This becomes even more important as systems grow.

## 12. Avoid boolean flag arguments

A boolean flag often means one function does two things.

Bad:

```python
def save_user(user: User, send_email: bool) -> None:
    ...
```

Better:

```python
def save_user(user: User) -> None:
    ...

def save_user_and_notify(user: User) -> None:
    ...
```

## 13. Keep conditionals simple

Complex nested `if` blocks are hard to reason about.

Bad:

```python
if user:
    if user.is_active:
        if user.email:
            send_email(user)
```

Better:

```python
if not user:
    return

if not user.is_active:
    return

if not user.email:
    return

send_email(user)
```

Use guard clauses to reduce nesting.

## 14. Replace condition-heavy logic with mapping or polymorphism when appropriate

Bad:

```python
def calculate_discount(user_type: str) -> int:
    if user_type == "regular":
        return 0
    if user_type == "premium":
        return 10
    if user_type == "vip":
        return 20
    raise ValueError("Unknown user type")
```

Better:

```python
DISCOUNT_BY_USER_TYPE: dict[str, int] = {
    "regular": 0,
    "premium": 10,
    "vip": 20,
}
```

## 15. Errors should be explicit

Do not silently ignore problems.
Raise meaningful exceptions.

Bad:

```python
try:
    process_payment(payment)
except Exception:
    pass
```

Better:

```python
try:
    process_payment(payment)
except PaymentProviderError as exc:
    raise PaymentFailedError("Payment processing failed") from exc
```

## 16. Use custom exceptions for business rules

Business errors should be named after the domain.

```python
class OrderAlreadyPaidError(Exception):
    pass


class ProductOutOfStockError(Exception):
    pass
```

And use dedicated checks when it improves readability:

```python
def _raise_if_order_already_paid(order: Order) -> None:
    if order.is_paid:
        raise OrderAlreadyPaidError(order.id)
```

This keeps business rules explicit, reusable, and easy to locate.

## 17. Separate business logic from frameworks

Core logic should not depend too much on FastAPI, Django, SQLAlchemy, or CLI details.

Bad:

```python
def create_order(request: Request, db: Session):
    ...
```

Better:

```python
def create_order(command: CreateOrderCommand, repository: OrderRepository) -> Order:
    ...
```

Frameworks should stay near the edges.

## 18. Keep side effects controlled

A function should either:

* compute and return a result
* or perform a side effect

Mixing both too much makes code harder to test.

## 19. Avoid duplication, but do not over-abstract

Do not repeat the same logic in many places.
But do not create abstractions too early.

Rule:

* remove duplication of knowledge
* tolerate small duplication if abstraction would make code worse

## 20. Organize code by feature, not only by type

Instead of this:

```text
models/
services/
repositories/
schemas/
```

Prefer something like:

```text
users/
    dto.py
    service.py
    models.py
    repository.py
orders/
    dto.py
    service.py
    models.py
    repository.py
```

This keeps related code together.

## 21. Keep classes small too

A class should have a focused purpose.
If a class has too many methods or too many reasons to change, split it.

Bad signs:

* too many fields
* too many dependencies
* methods unrelated to each other

## 22. Write tests that read like documentation

A good test explains expected behavior.

Bad:

```python
def test_1():
    ...
```

Better:

```python
def test_create_order_raises_error_when_cart_is_empty():
    ...
```

Test behavior, not implementation details.

## 23. Consistency beats personal style

A codebase should feel like it was written by one person.

Be consistent in:

* naming
* folder structure
* DTO usage
* typing
* error handling
* logging
* return style

## 24. Use typing to make contracts obvious

Types should make the code easier to understand at first glance.

Good:

```python
def calculate_total(prices: list[float]) -> float:
    return sum(prices)
```

Better:

```python
def calculate_order_total(items: list[OrderItem]) -> Money:
    return sum(item.total_price for item in items)
```

Use types to communicate intent, not just satisfy tooling.

## 25. Prefer readability over micro-optimization

Do not make code harder to read for tiny performance gains unless performance is proven to matter.

First make it correct. Then make it clean. Then optimize if needed.

## 26. Leave the code better than you found it

Whenever you touch code:

* rename one bad variable
* split one long function
* add one missing type
* replace one raw dict with a DTO
* improve one test
* remove one duplication

Small improvements compound.


Here is a clean extra section you can append to the guideline.

---

## 27. Do not bypass quality tools without a very strong reason

Do not treat linters, type checkers, and static analysis as obstacles to silence.
Treat them as feedback tools that help keep the codebase healthy.

Avoid using things like:

* `if TYPE_CHECKING:` as a routine escape hatch
* `# noqa`
* `# type: ignore`
* `# pylint: disable=...`
* `# pragma: no cover`
* broad tool-level ignores in config
* fake abstractions created only to satisfy tooling without improving design

These mechanisms are sometimes necessary, but they should be **rare exceptions**, not normal practice.

### Why this matters

Every bypass weakens the contract of the code.

When developers constantly suppress warnings instead of fixing root causes, the codebase becomes:

* harder to trust
* harder to refactor
* harder to navigate
* easier to break silently

If the tool reports a problem, the first question should be:

**“What is wrong with the code or design?”**

not:

**“How do I silence this warning?”**

---

## 28. `if TYPE_CHECKING:` should not hide poor structure

`if TYPE_CHECKING:` can be useful in rare cases, but it often appears because the code has structural problems:

* circular imports
* tangled module boundaries
* weak dependency direction
* domain and infrastructure mixed together

Bad:

```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from app.users.service import UserService
```

If this appears often, it is usually a sign that imports and module responsibilities need to be reorganized.

Prefer:

* extracting shared interfaces or DTOs into a lower-level module
* separating domain models from infrastructure code
* reducing coupling between modules
* moving common types into dedicated modules like `dto.py`, `types.py`, or `interfaces.py`

Use `if TYPE_CHECKING:` only when there is a real technical need, not as a standard architectural pattern.

---

## 29. Do not hide type problems with `# type: ignore`

`# type: ignore` should be a last resort.

Bad:

```python
result = some_untyped_library_call()  # type: ignore
```

Better:

* write an explicit adapter
* narrow the type properly
* add a protocol or interface
* introduce a typed wrapper
* cast only where the contract is actually known

Better example:

```python
from typing import cast

raw_user = external_client.get_user()
user = cast(UserDTO, raw_user)
```

Even `cast()` should be used carefully, but it is still better than silently suppressing the checker without explanation.

If suppression is truly unavoidable, it should be:

* local
* specific
* documented

Bad:

```python
value = get_value()  # type: ignore
```

Less bad:

```python
value = get_value()  # type: ignore[arg-type]  # third-party stub is incorrect here
```

Silence only the exact issue, and explain why.

---

## 30. Do not hide lint problems with `# noqa`

`# noqa` is often used to keep bad code instead of improving it.

Bad:

```python
from app.users.service import UserService, UserManager, UserFactory  # noqa
```

Ask instead:

* Is this import really needed here?
* Is the module doing too much?
* Is the line too long because the code is badly structured?
* Is the naming too vague or repetitive?

Fix the code first. Suppress only when the warning is truly wrong or unavoidable.

The same applies to:

* `# pylint: disable=...`
* `ruff: noqa`
* file-level ignore comments
* broad ignore rules in config

A suppressed warning should feel unusual and slightly uncomfortable.

---

## 31. Do not use `# pragma: no cover` to avoid testing hard code

If code is hard to test, that is often a design signal.

Bad:

```python
def process_payment() -> None:
    ...
    except Exception:  # pragma: no cover
        ...
```

Ask instead:

* Why is this hard to test?
* Is the function doing too much?
* Are side effects mixed with business logic?
* Should dependencies be injected?
* Can error handling be extracted?

Coverage exclusions are acceptable for truly unreachable or environment-specific code, but not for ordinary business paths that are simply inconvenient to test.

---

## 32. Never normalize bypassing tools

Do not let the codebase develop habits like:

* “just add `noqa`”
* “just ignore typing here”
* “just disable the rule”
* “just exclude it from coverage”

That culture slowly destroys code quality.

A healthy rule is:

**Every ignore must justify itself.
No ignore should exist just because fixing the code is inconvenient.**

---

## 33. Prefer solving root causes

When a tool complains, prefer these fixes:

* restructure imports
* split modules
* introduce DTOs
* add explicit types
* define protocols
* wrap untyped libraries
* simplify functions
* remove dead code
* make dependencies directional and clear

The goal is not to make the linter quiet.

The goal is to make the code correct, clear, and easy to trust.

---

## Practical rule

If you need one of these:

* `if TYPE_CHECKING:`
* `# noqa`
* `# type: ignore`
* `# pragma: no cover`

stop and ask:

**Is this solving a real problem, or hiding one?**

If it is hiding one, fix the design instead.

---

And here is a shorter version in one sentence, if you want to insert it into the checklist style:

**Do not bypass linters, type checkers, or coverage tools unless it is truly unavoidable; repeated use of `if TYPE_CHECKING`, `noqa`, `type: ignore`, or `pragma: no cover` usually signals design problems that should be fixed at the root.**


Here is a clean section you can append as well.

---

## 34. Avoid local imports inside functions

Imports should normally be placed at the **top of the module**, not inside functions, methods, or branches.

Bad:

```python
def create_user_service() -> UserService:
    from app.users.service import UserService
    return UserService()
```

Bad:

```python
def process_order(order: Order) -> None:
    if order.is_external:
        from app.integrations.external import send_order
        send_order(order)
```

Better:

```python
from app.integrations.external import send_order
from app.users.service import UserService


def create_user_service() -> UserService:
    return UserService()


def process_order(order: Order) -> None:
    if order.is_external:
        send_order(order)
```

---

## 35. Why local imports are harmful

Local imports are often not a solution.
They are often a way to hide structural problems.

Usually they appear because of:

* circular imports
* bad module boundaries
* mixed responsibilities
* dependency direction issues
* attempts to avoid fixing architecture properly

They make code:

* harder to read
* harder to reason about
* harder to analyze statically
* harder to test
* more surprising for other developers

When reading a module, a developer should be able to understand its dependencies from the top of the file.

Imports are part of the module contract.
Do not hide them in the middle of logic.

---

## 36. Local imports usually signal a design problem

If you need a local import, ask:

* why do these modules depend on each other?
* should shared types be extracted into a separate module?
* are domain and infrastructure too tightly coupled?
* does this class know too much?
* should I invert the dependency with an interface or protocol?

Very often the right fix is one of these:

* split the module
* move shared DTOs or types into a lower-level module
* extract interfaces
* reduce coupling
* reorganize feature boundaries

Bad:

```python
# users/service.py
def create_user(...) -> None:
    from app.notifications.service import send_welcome_email
    ...
```

```python
# notifications/service.py
def send_welcome_email(...) -> None:
    from app.users.service import get_user_email
    ...
```

This is not an import problem.
This is an architecture problem.

---

## 37. Keep dependencies visible and explicit

A clean module should show its dependencies immediately.

Good:

```python
from app.notifications.interfaces import NotificationSender
from app.users.dto import CreateUserRequest, UserResponse
from app.users.repository import UserRepository
```

This is much better than forcing the reader to discover dependencies later inside method bodies.

Visible imports improve:

* readability
* predictability
* static analysis
* maintainability

---

## 38. Rare exceptions may exist, but they should stay rare

A local import may be acceptable in narrow technical cases, for example:

* optional dependency that is not always installed
* heavy dependency used only in a very specific execution path
* startup-sensitive code where import cost truly matters
* temporary compatibility boundary with third-party code

But even in these cases:

* keep it explicit
* document why it is local
* do not make it the default style
* do not use it to hide circular imports

Bad:

```python
def export_report(report: Report) -> bytes:
    from some_library import export
    return export(report)
```

Less bad:

```python
def export_report(report: Report) -> bytes:
    # Local import because this dependency is optional and only needed for PDF export.
    from some_library import export

    return export(report)
```

If there is no strong technical reason, the import belongs at module level.

---

## 39. Practical rule

**If you need a local import, first assume the code structure is wrong.
Fix the dependency design before choosing the import workaround.**

---

And here is a short checklist-style sentence for insertion into the main guideline:

**Avoid local imports inside functions and methods; they usually hide circular dependencies or poor module boundaries, and dependencies should stay visible at the top of the file.**


---

# Practical checklist

Before you commit, ask:

* Is every name clear?
* Does each function do one thing?
* Is every function argument typed?
* Is every function return typed?
* Am I using DTOs instead of raw dictionaries?
* Are business rules explicit?
* Are errors handled clearly?
* Is duplication justified?
* Can a new developer understand this quickly?

---

# Golden rule

**Clean code is code that is easy to read, easy to change, and hard to misuse.**

I can also turn this into a **shorter team-ready `.md` document** with a more strict, policy-like style.
