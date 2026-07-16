# DeployReady — Kora Analytics

A containerised Node.js API with an automated test → build → push → deploy pipeline, deployed using Docker and Render.

---

# Architecture Overview

```
 GitHub push to main
        │
        ▼
 ┌────────────────────┐
 │ GitHub Actions      │
 │                    │
 │ 1. npm test         │  <- stops if tests fail
 │ 2. docker build     │
 │ 3. push image       │
 └─────────┬──────────┘
           │
           ▼
 ┌────────────────────┐
 │ Render              │
 │                    │
 │ Docker container    │
 │ Node.js API         │
 │ PORT=3000           │
 └─────────┬──────────┘
           │
           ▼
        Users
```

---

# Application

The application is located inside the `app/` directory.

It is an Express.js API with three endpoints:

| Method | Route | Description |
|---|---|---|
| GET | `/health` | Returns application health status |
| GET | `/metrics` | Returns uptime and memory usage |
| POST | `/data` | Accepts JSON data and returns it back |

Example:

### GET `/health`

Response:

```json
{
  "status": "ok"
}
```

---

# Project Structure

```
DeployReady/

├── app/
│   ├── server.js
│   ├── package.json
│   ├── package-lock.json
│   └── test/
│       └── test_api.js
│
├── .github/
│   └── workflows/
│       └── deploy.yml
│
├── Dockerfile
├── docker-compose.yml
├── .env.example
├── DEPLOYMENT.md
└── README.md
```

---

# Containerisation

The application is containerised using Docker.

The Docker setup:

- Uses the lightweight `node:20-alpine` image
- Installs Node.js dependencies
- Accepts the `PORT` environment variable
- Runs the application as a non-root user
- Starts the Express API inside the container

The application listens on:

```
PORT=3000
```

---

# Running Locally Without Docker

Navigate to the application folder:

```bash
cd app
```

Install dependencies:

```bash
npm install
```

Start the application:

```bash
npm start
```

The API will run on:

```
http://localhost:3000
```

Test the health endpoint:

```
http://localhost:3000/health
```

Expected response:

```json
{
  "status": "ok"
}
```

---

# Running Tests

The project uses Node.js built-in testing.

Run:

```bash
cd app
npm test
```

The tests verify:

- `/health` returns the correct response
- `/metrics` returns uptime and memory usage
- `/data` returns the submitted JSON body

---

# Running With Docker Compose

Create your environment file:

```bash
cp .env.example .env
```

Build and start the container:

```bash
docker compose up --build
```

The API will be available at:

```
http://localhost:3000/health
```

---

# CI/CD Pipeline

The project uses GitHub Actions for continuous integration and deployment.

Workflow file:

```
.github/workflows/deploy.yml
```

The pipeline runs automatically whenever code is pushed to the `main` branch.

## Continuous Integration (CI)

The CI process checks that the application works before deployment.

Steps:

### 1. Install dependencies

The pipeline installs Node.js packages.

### 2. Run automated tests

The pipeline runs:

```bash
npm test
```

If tests fail, the pipeline stops and deployment does not continue.

---

## Continuous Deployment (CD)

After successful tests:

### 1. Build Docker Image

GitHub Actions builds the Docker image.

The image is tagged using the Git commit SHA.

Example:

```
deployready-app:<commit-sha>
```

### 2. Push Image

The Docker image is pushed to GitHub Container Registry.

### 3. Deploy Application

The application is deployed using Render's Docker-based hosting platform.

Render automatically builds and runs the container.

---

# Cloud Deployment

## Cloud Provider

This project uses:

```
Render
```

Render was selected because it provides Docker-based cloud deployment without requiring manual server management.

Benefits:

- Supports Docker containers
- Automatic deployments
- Environment variable management
- Automatic application restarts
- Simple cloud hosting setup

---

# Deployment Process

The deployment process is:

```
Developer
    |
    |
git push
    |
    ↓
GitHub Repository
    |
    ↓
GitHub Actions
    |
    ↓
Run Tests
    |
    ↓
Build Docker Image
    |
    ↓
Push Image
    |
    ↓
Render Deployment
    |
    ↓
Running API
```

---

# Environment Variables

The project uses environment variables for configuration.

Example:

```
PORT=3000
```

A template file is included:

```
.env.example
```

Real environment files are not committed to GitHub.

---

# Deployment Verification

After deployment, the API health endpoint can be checked:

```
https://<your-render-url>/health
```

Expected response:

```json
{
  "status": "ok"
}
```

---

# Design Decisions

## Docker

Docker was used to package the application and its dependencies into a consistent environment.

This allows the application to run the same way locally and in the cloud.

---

## GitHub Actions

GitHub Actions was selected to automate testing, building, and deployment.

This removes manual deployment steps and improves reliability.

---

## GitHub Container Registry

GitHub Container Registry was chosen because it integrates directly with GitHub Actions and provides secure Docker image storage.

---

## Render Deployment

Render was selected because it provides managed Docker deployment without requiring manual server configuration.

This allows focusing on the application and DevOps workflow instead of server maintenance.

---

## Testing

Node.js built-in testing (`node --test`) was used because it is included with modern Node.js versions and avoids unnecessary dependencies.

---

# Future Improvements

Possible improvements include:

- Add application monitoring and alerts
- Add automatic rollback after failed deployments
- Add Infrastructure as Code using Terraform
- Add more API test coverage
- Add security scanning for Docker images