# FastAPI Calculator Application with Database Integration

A production-ready FastAPI application featuring a calculator API with comprehensive database integration using SQLAlchemy ORM, PostgreSQL, and a complete CI/CD pipeline.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Installation](#installation)
- [Running the Application](#running-the-application)
- [Database Models](#database-models)
- [Pydantic Schemas](#pydantic-schemas)
- [Design Patterns](#design-patterns)
- [Testing](#testing)
- [CI/CD Pipeline](#cicd-pipeline)
- [Docker Deployment](#docker-deployment)
- [API Documentation](#api-documentation)

## Overview

This project implements a FastAPI-based calculator application with full database persistence using SQLAlchemy ORM and PostgreSQL. It demonstrates advanced software engineering practices including polymorphic inheritance, factory patterns, comprehensive testing, and automated CI/CD deployment to Docker Hub.

The application supports four arithmetic operations (addition, subtraction, multiplication, division) with full database tracking, user management, and calculation history.

## Features

- **RESTful Calculator API**: Four basic arithmetic operations via HTTP endpoints
- **Database Persistence**: SQLAlchemy ORM with PostgreSQL for storing calculations and users
- **Polymorphic Inheritance**: Elegant calculation model hierarchy using SQLAlchemy's polymorphic features
- **Factory Pattern**: Clean object creation for different calculation types
- **Pydantic Validation**: Comprehensive request/response validation with custom validators
- **Multi-Layer Testing**: Unit, integration, and end-to-end tests with >90% coverage
- **CI/CD Pipeline**: Automated testing, security scanning, and Docker deployment
- **Production-Ready**: Docker containerization with health checks and security best practices

## Architecture

The application follows a layered architecture:

```
├── Presentation Layer (FastAPI Routes)
├── Validation Layer (Pydantic Schemas)
├── Business Logic Layer (Operations)
├── Data Access Layer (SQLAlchemy Models)
└── Database Layer (PostgreSQL)
```

Key architectural decisions:

- **Polymorphic Single Table Inheritance**: All calculation types stored in one table with discriminator column
- **Factory Pattern**: Centralized calculation object creation
- **Repository Pattern**: Database access abstraction through SQLAlchemy
- **Dependency Injection**: FastAPI's dependency system for database sessions

## Technology Stack

- **Framework**: FastAPI 0.115.4
- **Database**: PostgreSQL (via Docker)
- **ORM**: SQLAlchemy 2.0.38
- **Validation**: Pydantic 2.9.2
- **Testing**: pytest 8.3.3, pytest-cov 6.0.0, Playwright 1.48.0
- **Web Server**: Uvicorn 0.32.0
- **Templating**: Jinja2 3.1.4
- **Containerization**: Docker with multi-platform builds
- **CI/CD**: GitHub Actions
- **Security**: Trivy vulnerability scanner

## Project Structure

```
.
├── .github/
│   └── workflows/
│       └── python-app.yml          # CI/CD pipeline configuration
├── app/
│   ├── core/
│   │   ├── __init__.py
│   │   └── config.py               # Application settings
│   ├── models/
│   │   ├── __init__.py
│   │   ├── calculation.py          # Polymorphic calculation models
│   │   └── user.py                 # User model
│   ├── operations/
│   │   └── __init__.py             # Basic arithmetic operations
│   ├── schemas/
│   │   ├── __init__.py
│   │   └── calculation.py          # Pydantic validation schemas
│   └── database.py                 # Database configuration
├── tests/
│   ├── unit/                       # Unit tests
│   ├── integration/                # Integration tests
│   ├── e2e/                        # End-to-end tests
│   └── conftest.py                 # Test fixtures
├── templates/
│   └── index.html                  # Frontend interface
├── Dockerfile                      # Production container definition
├── docker-compose.yml              # Local development setup
├── requirements.txt                # Python dependencies
├── pytest.ini                      # Test configuration
├── main.py                         # Application entry point
└── README.md                       # This file
```

## Installation

### Prerequisites

- Python 3.10 or higher
- PostgreSQL 13 or higher (or Docker)
- Docker and Docker Compose (optional, for containerized setup)
- Git

### Local Setup

1. Clone the repository:
```bash
git clone <repository-url>
cd <repository-name>
```

2. Create and activate a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements.txt
```

4. Install Playwright browsers (for E2E tests):
```bash
playwright install
```

5. Set up environment variables (optional):
Create a `.env` file in the root directory:
```env
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/calculator_db
```

## Running the Application

### Using Docker Compose (Recommended)

```bash
docker-compose up
```

This starts:
- FastAPI application on `http://localhost:8000`
- PostgreSQL database on port 5432
- pgAdmin on `http://localhost:5050`

### Local Development

1. Start PostgreSQL (if not using Docker):
```bash
# Example using local PostgreSQL
createdb calculator_db
```

2. Run the application:
```bash
uvicorn main:app --reload
```

3. Access the application:
- Web Interface: `http://localhost:8000`
- API Documentation: `http://localhost:8000/docs`
- Alternative API Docs: `http://localhost:8000/redoc`

## Database Models

### User Model

Located in `app/models/user.py`, the User model represents system users:

```python
class User(Base):
    id: UUID (Primary Key)
    username: String(50, unique, indexed)
    email: String(100, unique, indexed)
    created_at: DateTime
    updated_at: DateTime
    calculations: Relationship (one-to-many with Calculation)
```

### Calculation Model (Polymorphic Inheritance)

Located in `app/models/calculation.py`, implements Single Table Inheritance:

**Base Class: Calculation**
- Stores all calculation types in one table
- Uses `type` column as discriminator
- Common fields: `id`, `user_id`, `type`, `inputs`, `result`, `created_at`, `updated_at`

**Subclasses:**
- `Addition`: Sums all input numbers
- `Subtraction`: Sequential subtraction
- `Multiplication`: Product of all inputs
- `Division`: Sequential division with zero-checking

**Key Features:**
- UUID primary keys for distributed systems
- JSON column for flexible input storage
- Automatic timestamp management
- Cascade delete from User to Calculations
- Factory method for object creation

Example usage:
```python
# Factory pattern
calc = Calculation.create('addition', user_id, [10, 20, 30])
result = calc.get_result()  # Returns 60

# Polymorphic query
all_calcs = session.query(Calculation).all()  # Returns correct subclass instances
```

## Pydantic Schemas

Located in `app/schemas/calculation.py`, schemas provide validation at the API boundary:

### CalculationType Enum
```python
class CalculationType(str, Enum):
    ADDITION = "addition"
    SUBTRACTION = "subtraction"
    MULTIPLICATION = "multiplication"
    DIVISION = "division"
```

### CalculationBase
Base schema with common validation:
- Type validation (case-insensitive)
- Input validation (minimum 2 numbers)
- Division by zero prevention
- Cross-field validation

### CalculationCreate
Extends CalculationBase for creation requests:
- Requires `type`, `inputs`, and `user_id`
- Validates UUID format
- Business rule enforcement

### CalculationUpdate
Partial update schema:
- All fields optional
- Maintains validation rules

### CalculationResponse
Output serialization schema:
- Includes all fields plus computed result
- Timestamp serialization
- Database model compatibility via `from_attributes=True`

## Design Patterns

### Factory Pattern

The `Calculation.create()` method implements the Factory Pattern:

**Benefits:**
- Centralized object creation logic
- Easy to extend with new calculation types
- Type-safe instantiation
- Encapsulates complexity

**Implementation:**
```python
@classmethod
def create(cls, calculation_type: str, user_id: uuid.UUID, 
           inputs: List[float]) -> "Calculation":
    calculation_classes = {
        'addition': Addition,
        'subtraction': Subtraction,
        'multiplication': Multiplication,
        'division': Division,
    }
    calculation_class = calculation_classes.get(calculation_type.lower())
    if not calculation_class:
        raise ValueError(f"Unsupported calculation type: {calculation_type}")
    return calculation_class(user_id=user_id, inputs=inputs)
```

### Repository Pattern

Database access is abstracted through SQLAlchemy sessions with dependency injection:

```python
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### Template Method Pattern

Abstract base class defines structure, subclasses provide implementation:

```python
class AbstractCalculation:
    def get_result(self) -> float:
        raise NotImplementedError("Subclasses must implement get_result()")

class Addition(Calculation):
    def get_result(self) -> float:
        return sum(self.inputs)
```

## Testing

### Test Structure

The project includes three layers of testing:

1. **Unit Tests** (`tests/unit/`): Test individual functions in isolation
2. **Integration Tests** (`tests/integration/`): Test component interactions
3. **End-to-End Tests** (`tests/e2e/`): Test full user workflows with Playwright

### Running Tests Locally

**All Tests:**
```bash
pytest
```

**With Coverage Report:**
```bash
pytest --cov=app --cov-report=html
```

**Specific Test Categories:**
```bash
pytest tests/unit/                    # Unit tests only
pytest tests/integration/             # Integration tests only
pytest tests/e2e/ -m e2e             # E2E tests only
```

**Coverage Report:**
After running tests with coverage, open `htmlcov/index.html` in your browser.

### Test Configuration

Configuration in `pytest.ini`:
- Coverage target: `app` module
- Test markers: `slow`, `fast`, `e2e`
- Warning suppression for clean output

### Key Test Files

**Unit Tests:**
- `tests/unit/test_calculator.py`: Arithmetic operations
  - Parameterized tests for all operations
  - Edge cases (negatives, floats, zeros)
  - Error handling (division by zero)

**Integration Tests:**
- `tests/integration/test_calculation.py`: Polymorphic model behavior
  - Factory pattern validation
  - Type resolution verification
  - Input validation
  - Polymorphic querying

- `tests/integration/test_calculation_schema.py`: Pydantic validation
  - Valid/invalid data handling
  - Business rule enforcement
  - Cross-field validation
  - Error message verification

- `tests/integration/test_fastapi_calculator.py`: API endpoints
  - HTTP request/response validation
  - Status code verification
  - Error handling

**End-to-End Tests:**
- `tests/e2e/test_e2e.py`: Browser automation
  - Full user workflows
  - Frontend interaction
  - Server integration

### Test Coverage

Current coverage: >90% across all modules

Key coverage areas:
- Models: 100% (polymorphic behavior, factory pattern)
- Schemas: 98% (validation logic, business rules)
- Operations: 100% (arithmetic functions)
- API Endpoints: 95% (happy paths and error cases)

## CI/CD Pipeline

### GitHub Actions Workflow

The CI/CD pipeline (`.github/workflows/python-app.yml`) consists of three jobs:

#### 1. Test Job

**Purpose:** Run all tests with PostgreSQL service

**Steps:**
1. Checkout code
2. Setup Python 3.13
3. Cache dependencies
4. Install dependencies and Playwright
5. Run unit tests with coverage
6. Run integration tests
7. Run E2E tests

**Services:**
- PostgreSQL 13 with health checks
- Database: `myappdb`
- Credentials: `user/password`

#### 2. Security Job

**Purpose:** Scan Docker image for vulnerabilities

**Steps:**
1. Checkout code
2. Build Docker image
3. Run Trivy security scanner
4. Check for CRITICAL and HIGH severity vulnerabilities

**Configuration:**
- Exit on vulnerabilities: Yes
- Ignore unfixed: Yes
- Ignored CVEs: Listed in `.trivyignore`

#### 3. Deploy Job

**Purpose:** Build and push multi-platform Docker image

**Conditions:**
- Only runs on `main` branch
- Requires test and security jobs to pass
- Uses `production` environment

**Steps:**
1. Checkout code
2. Setup Docker Buildx
3. Login to Docker Hub
4. Build and push image with tags:
   - `latest`
   - Git commit SHA
5. Multi-platform build: `linux/amd64`, `linux/arm64`
6. Registry caching for faster builds

**Required Secrets:**
- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`

### Setting Up CI/CD

1. **Configure GitHub Secrets:**
   - Navigate to repository Settings > Secrets and variables > Actions
   - Add `DOCKERHUB_USERNAME` (your Docker Hub username)
   - Add `DOCKERHUB_TOKEN` (Docker Hub access token, not password)

2. **Create Production Environment:**
   - Go to Settings > Environments
   - Create `production` environment
   - Optionally add protection rules

3. **Update Docker Hub Repository:**
   - Change `cerechukwu27/module11` in workflow to your repository name

### Monitoring Pipeline

View workflow runs:
- Go to Actions tab in GitHub repository
- Each push/PR triggers the workflow
- Green checkmark indicates success
- Click on runs for detailed logs

## Docker Deployment

### Docker Hub Repository

**Image Location:** `cerechukwu27/module11`

**Available Tags:**
- `latest`: Most recent build from main branch
- `<commit-sha>`: Specific commit versions

### Pulling the Image

```bash
docker pull cerechukwu27/module11:latest
```

### Running the Container

**Basic Run:**
```bash
docker run -p 8000:8000 cerechukwu27/module11:latest
```

**With Database Connection:**
```bash
docker run -p 8000:8000 \
  -e DATABASE_URL=postgresql://user:password@host:5432/dbname \
  cerechukwu27/module11:latest
```

**Using Docker Compose:**
```yaml
version: '3.8'
services:
  app:
    image: cerechukwu27/module11:latest
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://postgres:postgres@db:5432/calculator_db
    depends_on:
      - db
  
  db:
    image: postgres:latest
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: calculator_db
```

### Dockerfile Details

**Base Image:** `python:3.10-slim`

**Security Features:**
- Non-root user (`appuser`)
- Minimal attack surface
- Updated system packages
- No unnecessary tools

**Optimizations:**
- Multi-stage potential
- Layer caching
- Minimal dependencies
- Health check included

**Health Check:**
```dockerfile
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
   CMD curl -f http://localhost:8000/health || exit 1
```

## API Documentation

### Interactive Documentation

Once the application is running:

- **Swagger UI:** `http://localhost:8000/docs`
  - Interactive API testing
  - Request/response examples
  - Schema definitions

- **ReDoc:** `http://localhost:8000/redoc`
  - Alternative documentation view
  - Better for reading

### Available Endpoints

**Calculator Operations:**

- `POST /add` - Add two numbers
- `POST /subtract` - Subtract two numbers
- `POST /multiply` - Multiply two numbers
- `POST /divide` - Divide two numbers

**Request Format:**
```json
{
  "a": 10.5,
  "b": 5.2
}
```

**Success Response:**
```json
{
  "result": 15.7
}
```

**Error Response:**
```json
{
  "error": "Cannot divide by zero!"
}
```

### Web Interface

Navigate to `http://localhost:8000` for a simple web calculator interface with:
- Input fields for two numbers
- Buttons for each operation
- Result display
- Error handling

## Development Guidelines

### Code Style

- PEP 8 compliance
- Type hints throughout
- Comprehensive docstrings
- Meaningful variable names

### Adding New Calculation Types

1. Create new subclass in `app/models/calculation.py`:
```python
class Modulus(Calculation):
    __mapper_args__ = {"polymorphic_identity": "modulus"}
    
    def get_result(self) -> float:
        # Implementation
        pass
```

2. Add to factory dictionary in `Calculation.create()`

3. Update `CalculationType` enum in schemas

4. Write tests in `tests/integration/test_calculation.py`

### Database Migrations

For schema changes:
```bash
# Install Alembic
pip install alembic

# Initialize migrations
alembic init alembic

# Create migration
alembic revision --autogenerate -m "Description"

# Apply migration
alembic upgrade head
```

## Troubleshooting

### Common Issues

**Port Already in Use:**
```bash
# Find process using port 8000
lsof -i :8000
# Kill process
kill -9 <PID>
```

**Database Connection Failed:**
- Check PostgreSQL is running
- Verify DATABASE_URL is correct
- Ensure database exists
- Check network connectivity

**Tests Failing:**
- Install Playwright browsers: `playwright install`
- Check PostgreSQL service in CI
- Verify test database is clean
- Review test logs for details

**Docker Build Fails:**
- Clear Docker cache: `docker system prune -a`
- Check Dockerfile syntax
- Verify requirements.txt is complete
- Review build logs

### Getting Help

For issues:
1. Check logs: `docker-compose logs` or `uvicorn` output
2. Review test output: `pytest -v`
3. Verify environment variables
4. Check database connectivity
