# ✨ StringSphere API: Advanced String Processing & Natural Language Filtering

## Overview
StringSphere is a robust backend service built with **FastAPI** and **SQLModel**, designed for efficient storage, retrieval, and analytical processing of string data. It features advanced natural language processing (NLP) capabilities using **spaCy** to enable intuitive, human-like querying and filtering of stored strings based on their properties.

## Features
-   **String Management**: Perform CRUD operations (Create, Retrieve, Delete) on string entries.
-   **Automatic Property Calculation**: Automatically calculates properties like length, palindrome status, unique character count, word count, SHA256 hash, and character frequency map upon string creation.
-   **Structured Querying**: Filter strings based on properties like `is_palindrome`, `min_length`, `max_length`, `word_count`, and `contains_character` via standard API parameters.
-   **Natural Language Processing**: Leverage spaCy for interpreting natural language queries, allowing users to find strings using phrases like "strings longer than 10 characters" or "all single word palindromic strings."
-   **Robust Error Handling**: Comprehensive exception handling for request validation, data conflicts, and resource not found scenarios.
-   **CORS Enabled**: Configured with Cross-Origin Resource Sharing (CORS) to allow flexible integration with various client applications.

## Getting Started
To get this project up and running locally, follow these steps.

### Installation
Ensure you have Python 3.12 installed.
```bash
# Clone the repository
git clone https://github.com/yourusername/hng-stage-1.git # Replace with your actual repository URL
cd hng-stage-1

# Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate # On Windows, use `.venv\Scripts\activate`

# Install project dependencies
pip install -r requirements.txt

# Download the required spaCy model
python -m spacy download en_core_web_sm
```

### Environment Variables
This project uses a SQLite database (`database.db`) by default and does not require explicit environment variables for its current configuration. All necessary database parameters are hardcoded within `main.py`.

However, for production deployments, it is a best practice to configure database connections via environment variables. An example of how you *would* typically set a `DATABASE_URL` for a PostgreSQL database is shown below, though it is not used in this specific project's `main.py`.

```
# Example for a PostgreSQL database (not used in this project's code, but good practice)
# DATABASE_URL="postgresql://user:password@host:port/dbname"
```

## Usage
After installation, you can run the FastAPI application using Uvicorn.

```bash
uvicorn main:app --reload
```
The API documentation (Swagger UI) will then be available at `http://127.0.0.1:8000/docs`.

### Running with a Specific Python Version
The `.python-version` file indicates Python 3.12. If you use a tool like `pyenv`, it will automatically select this version.

```bash
# Example using pyenv
pyenv install 3.12.0
pyenv local 3.12.0
```

## API Documentation
### Base URL
`http://127.0.0.1:8000` (or your deployment URL)

### Endpoints
#### POST /strings
Adds a new string to the database, automatically calculating its properties.

**Request**:
```json
{
  "value": "example string"
}
```

**Response**:
```json
{
  "id": "a1b2c3d4-e5f6-7890-1234-567890abcdef",
  "value": "example string",
  "properties": {
    "length": 14,
    "is_palindrome": false,
    "unique_characters": 9,
    "word_count": 2,
    "sha256_hash": "a591a6d40bf420404a011733cfb7b190d62c65bf0bcda32b57b277d9ad9f146e",
    "character_frequency_map": {
      "e": 3,
      "x": 1,
      "a": 1,
      "m": 1,
      "p": 1,
      "l": 1,
      " ": 1,
      "s": 1,
      "t": 1,
      "r": 1,
      "i": 1,
      "n": 1,
      "g": 1
    }
  },
  "created_at": "2024-07-20T12:34:56.789Z"
}
```

**Errors**:
-   `400 Bad Request`: Invalid request body or missing 'value' field.
-   `409 Conflict`: String already exists in the system.
-   `422 Unprocessable Content`: Invalid datatype for 'value' (must be string).

#### GET /strings
Retrieves strings from the database based on query parameters.

**Request**:
Query parameters:
-   `is_palindrome`: `true` or `false`
-   `min_length`: `integer`
-   `max_length`: `integer`
-   `word_count`: `integer`
-   `contains_character`: `single_character` (e.g., 'a')

Example: `/strings?is_palindrome=true&min_length=5`

**Response**:
```json
{
  "data": [
    {
      "id": "uuid1",
      "value": "madam",
      "properties": { /* ... string properties ... */ },
      "created_at": "ISO-datetime"
    },
    {
      "id": "uuid2",
      "value": "level",
      "properties": { /* ... string properties ... */ },
      "created_at": "ISO-datetime"
    }
  ],
  "count": 2,
  "filters_applied": {
    "is_palindrome": true,
    "min_length": 5
  }
}
```

**Errors**:
-   `400 Bad Request`: Invalid query parameter values or types.
-   `422 Unprocessable Content`: Invalid query data type for a parameter.

#### GET /strings/{string_value}
Retrieves a specific string record by its value.

**Request**:
Path parameter `string_value`
Example: `/strings/hello`

**Response**:
```json
{
  "id": "a1b2c3d4-e5f6-7890-1234-567890abcdef",
  "value": "hello",
  "properties": {
    "length": 5,
    "is_palindrome": false,
    "unique_characters": 4,
    "word_count": 1,
    "sha256_hash": "2cf24dd0df9a63d91df2666a2e8739b8131343f115d909d737397b9195a9437b",
    "character_frequency_map": {
      "h": 1,
      "e": 1,
      "l": 2,
      "o": 1
    }
  },
  "created_at": "2024-07-20T12:34:56.789Z"
}
```

**Errors**:
-   `400 Bad Request`: Invalid request body or missing 'value' field.
-   `404 Not Found`: String does not exist in the system.
-   `422 Unprocessable Content`: Invalid datatype for 'value' (must be string).

#### DELETE /strings/{string}
Deletes a specific string record from the database.

**Request**:
Path parameter `string`
Example: `/strings/delete_me`

**Response**:
`204 No Content` (empty body)

**Errors**:
-   `400 Bad Request`: Invalid request body or missing 'value' field.
-   `404 Not Found`: String does not exist in the system.
-   `422 Unprocessable Content`: Invalid datatype for 'value' (must be string).

#### GET /strings/filter-by-natural-language
Filters strings using natural language queries.

**Request**:
Query parameter: `query="natural language string"`
Example: `/strings/filter-by-natural-language?query=all%20single%20word%20palindromic%20strings`

**Response**:
```json
{
  "data": [
    {
      "length": 5,
      "is_palindrome": true,
      "unique_characters": 3,
      "word_count": 1,
      "sha256_hash": "...",
      "character_frequency_map": { "m": 2, "a": 2, "d": 1 }
    }
  ],
  "count": 1,
  "filters_applied": {
    "original": "all single word palindromic strings",
    "parsed_filters": {
      "word_count": 1,
      "is_palindrome": true
    }
  }
}
```

**Errors**:
-   `400 Bad Request`: Unable to parse natural language query (e.g., query parameter missing or empty).
-   `422 Unprocessable Content`: Query parsed but resulting in conflicting or invalid filters.

## Technologies Used

| Technology | Description                                         | Link                                       |
| :--------- | :-------------------------------------------------- | :----------------------------------------- |
| Python     | Core programming language                           | [Python](https://www.python.org/)          |
| FastAPI    | High-performance web framework                      | [FastAPI](https://fastapi.tiangolo.com/)   |
| SQLModel   | Type-annotated ORM for SQL databases, built on Pydantic and SQLAlchemy | [SQLModel](https://sqlmodel.tiangolo.com/) |
| spaCy      | Advanced Natural Language Processing library        | [spaCy](https://spacy.io/)                 |
| Uvicorn    | ASGI web server for running FastAPI                 | [Uvicorn](https://www.uvicorn.org/)        |
| Pydantic   | Data validation and settings management             | [Pydantic](https://pydantic.dev/)          |

## Author Info

👋 **Fidelugwuowo Dilibe**
A passionate Python developer with a knack for building scalable and intelligent backend systems.

-   **LinkedIn**: [LinkedIn Profile](https://www.linkedin.com/in/dilibe-fidelugwuowo)
-   **Twitter**: [@therealdilibe](https://x.com/@therealdilibe)
-   **Portfolio**: [Portfolio Website](https://dilibe.netlify.app/)

---

[![Python 3.12](https://img.shields.io/badge/Python-3.12-blue?logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.119.1-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![SQLModel](https://img.shields.io/badge/SQLModel-0.0.27-red?logo=sqlmodel&logoColor=white)](https://sqlmodel.tiangolo.com/)
[![spaCy](https://img.shields.io/badge/spaCy-3.8.7-09A3D5?logo=spacy&logoColor=white)](https://spacy.io/)
[![Uvicorn](https://img.shields.io/badge/Uvicorn-0.38.0-F76023?logo=uvicorn&logoColor=white)](https://www.uvicorn.org/)

[![Readme was generated by Dokugen](https://img.shields.io/badge/Readme%20was%20generated%20by-Dokugen-brightgreen)](https://www.npmjs.com/package/dokugen)