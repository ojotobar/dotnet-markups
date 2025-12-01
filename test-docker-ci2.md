# PART 1 — UNIT TESTING TERMS & DEFINITIONS

## Core Testing Terminology

### 1. Test Case

A single test that verifies a specific behavior.

Example:

> "Login should fail if password is wrong"

---

### 2. Test Suite

A collection of related test cases.

Example:

> All tests related to authentication.

---

### 3. Assertion

A rule that must be true for the test to pass.

Examples:

* Result must not be null.
* Status must be 200 OK.
* A token must be returned.

---

### 4. Fixture

Setup code shared across multiple tests.

Example:

> Creating test database or sample users before tests run.

---

### 5. Test Runner

Program that runs tests and reports results.

Examples:

* `dotnet test`
* `xUnit`
* `NUnit`

---

### 6. Mock

A fake implementation of a dependency.

Used when:

* you don’t want real database
* you don’t want to send emails
* you don’t want real APIs

---

### 7. Stub

Simplified fake dependency that always returns the same data.

Difference:

* **Mock**: verifies calls
* **Stub**: just returns data

---

### 8. Code Coverage

How much of the code was executed by tests.

> High coverage = many lines tested
> Low coverage = many lines never tested

---

---

# STAGES / TYPES OF TESTING

## 1. Unit Testing

Tests individual methods/functions.

Characteristics:
✅ fast
✅ isolated
✅ no database
✅ no network
✅ no HTTP

---

## 2. Integration Testing

Tests communication between components.

Examples:

* Controller + service
* API + real database
* JWT generation + validation

---

## 3. System Testing

Tests the whole application like a user.

Example:

* Register → Login → Attendance → Logout

---

## 4. End-to-End (E2E) Testing

Simulates a real user.

Includes:

* UI
* API
* Database
* Network

---

## 5. Regression Testing

Verifies old features still work after changes.

---

## 6. Smoke Testing

Basic test to confirm:

> “Does the app start at all?”

---

## 7. Load Testing

Measures performance under pressure.

Examples:

* 1,000 logins at once
* 10,000 QR scans.

---

---

# COMMON TESTING METRICS

✅ Pass/fail rate
✅ Code coverage
✅ Execution time
✅ Flaky test count
✅ Failure frequency

---

# PART 2 — DOCKER TERMINOLOGY

## Core Docker Vocabulary

### Image

A packaged application template.

---

### Container

A running image.

---

### Dockerfile

A file that defines how to build an image.

---

### Registry

Image store (DockerHub, AWS ECR, Azure ACR).

---

### Docker Engine

The software that runs containers.

---

### Layer

Each Dockerfile step becomes a cached image layer.

---

### Volume

Persistent data storage.

---

### Bind Mount

Connecting local folders to container folders.

---

### Port Mapping

Exposing container port → host port.

---

### Tag

Version label for images.

Example:
`myapp:1.0`

---

---

# PART 3 — DOCKER COMMAND CHEAT SHEET

## Docker Basics

### List images

```bash
docker images
```

---

### List containers

```bash
docker ps
docker ps -a
```

---

### Build image

```bash
docker build -t myapp .
```

---

### Run container

```bash
docker run -p 8080:80 myapp
```

---

### Stop container

```bash
docker stop <container-id>
```

---

### Remove container

```bash
docker rm <container-id>
```

---

### Remove image

```bash
docker rmi myapp
```

---

### View logs

```bash
docker logs <container-id>
```

---

### Execute inside container

```bash
docker exec -it <container-id> /bin/bash
```

---

### Remove EVERYTHING (danger zone)

```bash
docker system prune -a
```

---

---

## Volume Commands

### Create volume

```bash
docker volume create mydata
```

---

### Attach volume

```bash
docker run -v mydata:/data myapp
```

---

### List volumes

```bash
docker volume ls
```

---

---

## Docker Compose

### Start services

```bash
docker compose up
```

---

### Rebuild

```bash
docker compose up --build
```

---

### Stop

```bash
docker compose down
```

---

---

# PART 4 — CI/CD TERMS & DEFINITIONS

---

## Core CI/CD Vocabulary

### Pipeline

Automated process that builds, tests, and deploys.

---

### Trigger

Event that starts a pipeline.

Example:

* push
* pull request
* merge

---

### Stage

Major pipeline step.

Examples:

* Build
* Test
* Deploy

---

### Job

Task run inside a stage.

Example:

> Run tests

---

### Step

Single command in a job.

---

### Runner / Agent

Machine that executes the pipeline.

---

### Artifact

Files produced by pipeline.

Examples:

* compiled binaries
* logs
* test results

---

### Fail-fast

Pipeline stops on failure.

---

### Environment

Deployment target.

Examples:

* dev
* staging
* prod

---

---

## Typical CI Tests

✅ Unit tests
✅ Integration tests
✅ Code analysis
✅ Security scanning
✅ Build validation

---

## Common CD Strategies

### Blue-Green Deployment

Two environments — switch traffic instantly.

---

### Rolling Deployment

Replace instances slowly.

---

### Canary Deployment

Release to few users first.

---

---

# PART 5 — TESTING TERMS BY TYPE (Quick Table)

| Term       | Meaning            |
| ---------- | ------------------ |
| Mock       | Fake dependency    |
| Stub       | Dummy data source  |
| Assertion  | Expected result    |
| Fixture    | Shared test setup  |
| Coverage   | Code tested        |
| Flaky Test | Inconsistent test  |
| Regression | Old features break |
| E2E        | Full flow tested   |
| Load       | Performance test   |

---

# PART 6 — COMMON INTERVIEW QUESTIONS (based on these notes)

✅ Difference between mock and stub?
✅ Difference between container and image?
✅ CI vs CD?
✅ What is a flaky test?
✅ When should integration testing be used?
✅ What does docker-compose do?
✅ What is an artifact?
✅ What is blue-green deployment?