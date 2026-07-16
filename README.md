# DeployReady — Kora Analytics

This project is a containerised Node.js API that I prepared to demonstrate DevOps practices including Docker, automated testing, CI/CD pipelines, and cloud deployment.

The goal of this project was to remove manual deployment steps by creating an automated workflow where code changes are tested, built into a Docker image, and deployed automatically.

---

# Project Architecture

```
Developer
    |
    | git push
    ↓
GitHub Repository
    |
    ↓
GitHub Actions

    1. Run tests
    2. Build Docker image
    3. Push Docker image

    |
    ↓

Render Cloud Platform

    |
    ↓

Running Node.js API

    |
    ↓

Users
```

---

# Application Overview

The application is located inside the `app/` directory.

It is a Node.js Express API with the following endpoints:

| Method | Route | Description |
|---|---|---|
| GET | `/health` | Checks if the application is running |
| GET | `/metrics` | Returns application uptime and memory usage |
| POST | `/data` | Receives JSON data and returns it back |

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

# Docker Containerisation

I used Docker to package the application and its dependencies into a container.

The Docker setup:

- Uses the `node:20-alpine` base image
- Installs the required Node.js dependencies
- Allows the application port to be configured using the `PORT` environment variable
- Runs the application using a non-root user for better security
- Starts the API inside the container

The application runs on:

```
PORT=3000
```

---

# Running the Application Locally

## Without Docker

First, move into the application folder:

```bash
cd app
```

Install dependencies:

```bash
npm install
```

Start the server:

```bash
npm start
```

The API will run at:

```
http://localhost:3000
```

To test:

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

# Running With Docker Compose

Create the environment file:

```bash
cp .env.example .env
```

Start the application:

```bash
docker compose up --build
```

The API will be available at:

```
http://localhost:3000/health
```

---

# Testing

I used Node.js built-in testing to verify that the API works correctly.

Run tests:

```bash
cd app
npm test
```

The tests check:

- The `/health` endpoint returns the correct status
- The `/metrics` endpoint returns uptime and memory information
- The `/data` endpoint correctly returns submitted JSON data

---

# CI/CD Pipeline

I created a GitHub Actions workflow to automate the deployment process.

The workflow is located at:

```
.github/workflows/deploy.yml
```

The pipeline runs automatically when I push changes to the `main` branch.

---

## Continuous Integration (CI)

The CI part makes sure that new changes do not break the application.

The pipeline:

1. Installs dependencies
2. Runs automated tests using:

```bash
npm test
```

If tests fail, the pipeline stops and the deployment process does not continue.

---

## Continuous Deployment (CD)

After successful tests, the pipeline continues with deployment.

The process:

1. Builds the Docker image
2. Tags the image using the Git commit SHA
3. Pushes the image to GitHub Container Registry
4. Deploys the application using Render

This removes the need for manually uploading files or restarting the application.

---

# Cloud Deployment

For deployment, I used **Render** as my cloud platform.

I selected Render because it supports Docker deployments and provides a simple way to run containerised applications without managing a virtual machine manually.

The deployment process:

```
GitHub Repository
        |
        ↓
GitHub Actions
        |
        ↓
Docker Image
        |
        ↓
Render Deployment
        |
        ↓
Live Application
```

---

# Environment Variables

The application uses environment variables instead of storing configuration directly inside the code.

Example:

```
PORT=3000
```

I included:

```
.env.example
```

as a template.

Sensitive information is not stored in the repository.

---

# My Implementation Decisions

## Docker

I used Docker because it allows the application to run consistently across different environments.

The same container can run locally and in the cloud.

---

## GitHub Actions

I used GitHub Actions to automate testing and deployment.

This reduces manual work and ensures that only tested code is deployed.

---

## GitHub Container Registry

I used GitHub Container Registry to store Docker images because it integrates well with GitHub Actions.

---

## Render

I chose Render because it provides an easy Docker deployment process without requiring paid cloud infrastructure setup.

It allows me to focus on the DevOps workflow while still deploying a real cloud application.

---

# Future Improvements

Some improvements I would add in the future:

- Add monitoring and alerts
- Add automatic rollback after failed deployments
- Add more API tests
- Use Infrastructure as Code tools such as Terraform
- Add security scanning for Docker images