# Python / FastAPI error handling guideline

## 1. Separate business errors from unexpected failures

Do not treat all errors the same.

**Business errors** are expected cases:

* user not found
* email already exists
* forbidden action
* invalid state transition

**Unexpected errors** are real failures:

* database is down
* external service timeout
* bug in code
* unhandled exception

Business errors should be raised intentionally.
Unexpected errors should be logged and returned as `500 Internal Server Error`.

---

## 2. Do not raise `HTTPException` in business logic

`HTTPException` should stay in the API layer.

Bad:

```python
from fastapi import HTTPException

def get_user(user_id: int) -> User:
    user = repo.get(user_id)
    if user is None:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

Better:

```python
class UserNotFoundError(Exception):
    pass


def get_user(user_id: int) -> User:
    user = repo.get(user_id)
    if user is None:
        raise UserNotFoundError("User not found")
    return user
```

---

## 3. Create your own exception hierarchy

Use custom exceptions for domain problems.

```python
class AppError(Exception):
    pass


class DomainError(AppError):
    pass


class NotFoundError(DomainError):
    pass


class ConflictError(DomainError):
    pass


class PermissionDeniedError(DomainError):
    pass


class UserNotFoundError(NotFoundError):
    pass


class EmailAlreadyExistsError(ConflictError):
    pass
```

This makes error handling predictable and easy to maintain.

---

## 4. Use dedicated `_raise_if_*` helpers for business rules

If a condition represents an important business rule, move it into a small helper.

```python
class OrderAlreadyPaidError(DomainError):
    pass


def _raise_if_order_already_paid(order: Order) -> None:
    if order.status == "paid":
        raise OrderAlreadyPaidError("Order is already paid")
```

Usage:

```python
def pay_order(order: Order) -> None:
    _raise_if_order_already_paid(order)
    order.status = "paid"
```

This helps because it:

* keeps service code clean
* makes rules reusable
* gives checks clear names

Good examples:

* `_raise_if_user_not_owner(...)`
* `_raise_if_order_closed(...)`
* `_raise_if_balance_too_low(...)`

---

## 5. Map exceptions to HTTP responses in one place

Use global FastAPI exception handlers.

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()


@app.exception_handler(NotFoundError)
async def handle_not_found(request: Request, exc: NotFoundError):
    return JSONResponse(
        status_code=404,
        content={"detail": str(exc)},
    )


@app.exception_handler(ConflictError)
async def handle_conflict(request: Request, exc: ConflictError):
    return JSONResponse(
        status_code=409,
        content={"detail": str(exc)},
    )


@app.exception_handler(Exception)
async def handle_unexpected_error(request: Request, exc: Exception):
    return JSONResponse(
        status_code=500,
        content={"detail": "Internal server error"},
    )
```

Do not repeat `try/except` in every endpoint.

---

## 6. Keep error responses consistent

Use one format everywhere.

Simple:

```json
{
  "detail": "User not found"
}
```

Better:

```json
{
  "error": {
    "code": "user_not_found",
    "message": "User not found"
  }
}
```

Clients should always know what error shape to expect.

---

## 7. Use machine-readable error codes

Do not rely only on free-text messages.

```json
{
  "error": {
    "code": "email_already_exists",
    "message": "Email already exists"
  }
}
```

This makes frontend and other clients easier to implement.

---

## 8. Log unexpected errors, not every business error

Expected domain errors usually do not need error-level logging.
Unexpected failures should be logged with traceback.

```python
import logging

logger = logging.getLogger(__name__)


@app.exception_handler(Exception)
async def handle_unexpected_error(request: Request, exc: Exception):
    logger.exception("Unhandled exception on %s", request.url.path)
    return JSONResponse(
        status_code=500,
        content={"detail": "Internal server error"},
    )
```

---

## 9. Do not leak internal details to clients

Do not return DB errors, tracebacks, or library internals in API responses.

Bad:

```json
{
  "detail": "psycopg2.errors.UniqueViolation: duplicate key value violates unique constraint"
}
```

Better:

```json
{
  "error": {
    "code": "email_already_exists",
    "message": "Email already exists"
  }
}
```

Internal details belong in logs.

---

## 10. Wrap low-level exceptions into meaningful application errors

Translate infrastructure exceptions into app-specific ones.

```python
from sqlalchemy.exc import IntegrityError


def create_user(email: str) -> User:
    try:
        return repo.create(email)
    except IntegrityError as exc:
        raise EmailAlreadyExistsError("Email already exists") from exc
```

Use `raise ... from exc` to preserve the original cause.

---

## 11. Keep endpoints thin

Endpoints should:

* receive request
* call service
* return response

Bad:

```python
@app.post("/users")
async def create_user(payload: CreateUserRequest):
    if await repo.exists_by_email(payload.email):
        raise HTTPException(status_code=409, detail="Email exists")
```

Better:

```python
@app.post("/users")
async def create_user(payload: CreateUserRequest, service: UserServiceDep):
    return await service.create_user(payload)
```

Business rules belong in services.

---

## 12. Test error paths too

Do not test only happy paths.

Also test:

* 404
* 409
* 403
* invalid business state
* unexpected 500

```python
def test_get_user_returns_404(client):
    response = client.get("/users/123")

    assert response.status_code == 404
```

---

# Recommended simple pattern

* **FastAPI / Pydantic** validate request shape
* **service layer** raises custom domain exceptions
* **`_raise_if_*` helpers** validate business conditions
* **global exception handlers** map exceptions to HTTP responses
* **unexpected exceptions** are logged and returned as `500`

## Minimal example

```python
class AppError(Exception):
    pass


class DomainError(AppError):
    pass


class NotFoundError(DomainError):
    pass


class UserNotFoundError(NotFoundError):
    pass


def _raise_if_user_not_found(user: User | None) -> None:
    if user is None:
        raise UserNotFoundError("User not found")
```

This is a clean default approach for most FastAPI projects.

I can also turn this into a clean `.md` document version for your project docs.
