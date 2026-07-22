# FastAPI End-to-End Learning Guide

This repository is a simple starting point for learning FastAPI from the ground up. The goal is to help you understand the full flow of building a FastAPI application:

- installing dependencies
- creating your first API
- defining routes and request/response models
- validating input with Pydantic
- running the app locally
- testing endpoints
- structuring a real project

## What is FastAPI?

FastAPI is a modern Python web framework for building APIs quickly with:

- high performance
- automatic OpenAPI docs
- type hints and data validation
- easy async support

It is commonly used for backend services, microservices, and internal APIs.

## Prerequisites

Before you begin, make sure you have:

- Python 3.10 or newer
- pip
- a terminal or VS Code integrated terminal

## Project Setup

Create and activate a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate
```

Install FastAPI and an ASGI server:

```bash
pip install fastapi "uvicorn[standard]"
```

Optional but useful for development:

```bash
pip install pydantic pytest httpx
```

## Your First FastAPI App

Create a file called `main.py`:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"message": "Hello from FastAPI!"}


@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id, "name": f"item-{item_id}"}
```

Run the app:

```bash
uvicorn main:app --reload
```

Open the browser at:

- `http://127.0.0.1:8000/`
- `http://127.0.0.1:8000/docs`

The `/docs` page is generated automatically by FastAPI and is one of the most useful features for learning.

## Understanding the Basics

### 1. App Creation

```python
from fastapi import FastAPI

app = FastAPI()
```

This creates the FastAPI application instance.

### 2. Route Decorators

```python
@app.get("/")
def read_root():
    return {"message": "Hello from FastAPI!"}
```

- `@app.get("/")` defines a GET endpoint
- the function returns JSON automatically

### 3. Path Parameters

```python
@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id}
```

FastAPI reads `item_id` from the URL and validates it as an integer.

## Request and Response Models

FastAPI works very well with Pydantic models.

Create a model:

```python
from typing import Optional
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    price: float
    is_offer: Optional[bool] = None
```

Use it in an endpoint:

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Optional

app = FastAPI()


class Item(BaseModel):
    name: str
    price: float
    is_offer: Optional[bool] = None


@app.post("/items/")
def create_item(item: Item):
    return {
        "message": "Item created",
        "item": item,
    }
```

This gives you:

- automatic request body validation
- clear error responses
- generated documentation for the request schema

## Query Parameters

Query parameters are passed in the URL:

```python
@app.get("/items/")
def list_items(skip: int = 0, limit: int = 10):
    return {"skip": skip, "limit": limit}
```

Example:

```text
/items/?skip=5&limit=20
```

## Status Codes

You can explicitly return HTTP status codes:

```python
from fastapi import status


@app.post("/items/", status_code=status.HTTP_201_CREATED)
def create_item(item: Item):
    return {"message": "Item created", "item": item}
```

## Response Models

You can define the shape of the response:

```python
from typing import List
from pydantic import BaseModel


class ItemResponse(BaseModel):
    name: str
    price: float


@app.get("/items/", response_model=List[ItemResponse])
def get_items():
    return [
        {"name": "Laptop", "price": 999.99},
        {"name": "Mouse", "price": 29.99},
    ]
```

This helps keep the API contract consistent.

## Basic Error Handling

FastAPI returns helpful validation errors automatically when the input is wrong.

Example request body:

```json
{
  "name": "Keyboard",
  "price": "not-a-number"
}
```

FastAPI will respond with a validation error describing what went wrong.

## Project Structure Idea

A more realistic project layout looks like this:

```text
app/
├── main.py
├── models.py
├── schemas.py
├── routes/
│   └── items.py
└── services/
    └── item_service.py
```

A simple version could be:

```text
main.py
app/
  __init__.py
  api/
    routes.py
```

## Example Full App

Here is a slightly larger example:

```python
from fastapi import FastAPI, status
from pydantic import BaseModel
from typing import Optional, List

app = FastAPI(title="My Learning API")


class Item(BaseModel):
    name: str
    price: float
    is_offer: Optional[bool] = None


items = []


@app.get("/")
def read_root():
    return {"message": "Welcome to the FastAPI learning app"}


@app.get("/items/", response_model=List[Item])
def get_items():
    return items


@app.post("/items/", status_code=status.HTTP_201_CREATED, response_model=Item)
def create_item(item: Item):
    items.append(item)
    return item
```

## Testing the API

You can test the API manually using the docs page or via CLI.

Example with `curl`:

```bash
curl http://127.0.0.1:8000/
curl -X POST "http://127.0.0.1:8000/items/" \
  -H "Content-Type: application/json" \
  -d '{"name":"Keyboard","price":49.99,"is_offer":true}'
```

### Example Test File

Create `test_main.py`:

```python
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)


def test_root():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json()["message"] == "Hello from FastAPI!"
```

Run tests:

```bash
pytest
```

## Common FastAPI Commands

Start the server:

```bash
uvicorn main:app --reload
```

Start on a specific port:

```bash
uvicorn main:app --host 0.0.0.0 --port 8001 --reload
```

Generate docs automatically:

- `/docs`
- `/redoc`

## Core Concepts to Learn Next

Once you are comfortable with the basics, focus on these topics:

1. dependency injection
2. authentication and authorization
3. database integration with SQLAlchemy or SQLModel
4. environment variables
5. background tasks
6. async endpoints
7. deployment to Docker, Render, Railway, or Azure

## Suggested Learning Path

1. Make a simple `GET /` endpoint
2. Add path parameters
3. Add query parameters
4. Use Pydantic models for request bodies
5. Add response models
6. Test with `TestClient`
7. Add a database layer
8. Containerize the app

## Notes

This repository is intentionally minimal so that you can build on top of it as you learn. You can start by adding new endpoints, schemas, and tests to practice.

## Next Step

Create a new Python file and experiment with these ideas:

- a `GET /users` endpoint
- a `POST /users` endpoint
- a `DELETE /users/{user_id}` endpoint
- a `GET /health` health check route
