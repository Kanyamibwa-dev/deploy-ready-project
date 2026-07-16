# DeployReady — Kora Analytics

This project demonstrates my understanding of DevOps practices by containerising a Node.js API, creating an automated CI/CD pipeline, and deploying the application to the cloud.

The main objective was to replace manual deployment steps with an automated process where code changes are tested, built into a Docker image, and deployed automatically.

---

## Architecture Overview

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
    3. Push image to GitHub Container Registry
    4. Trigger Render deployment

    |
    ↓

Render Cloud Platform

    |
    ↓

Running Node.js API
```

---

## Application

The application is a simple Node.js Express API located in the `app/` directory.

It provides three endpoints:

| Method | Route      | Description                     |
| ------ | ---------- | ------------------------------- |
| GET    | `/health`  | Returns application status      |
| GET    | `/metrics` | Returns uptime and memory usage |
| POST   | `/data`    | Receives and returns JSON data  |

Example:

```
GET /health
```

Response:

```json
{
  "status": "ok"
}
```

---

## Docker Containerisation

I used Docker to package the application and its dependencies into a container.

The Docker setup:

* Uses the `node:20-alpine` image
* Runs the application inside a container
* Accepts the `PORT` environment variable
* Runs using a non-root user for better security

The application runs on:

```
PORT=3000
```

To start the application using Docker:

```bash
docker compose up --build
```

---

## CI/CD Pipeline

I created a GitHub Actions workflow in:

```
.github/workflows/deploy.yml
```

The pipeline runs automatically when I push changes to the `main` branch.

The workflow performs these steps:

1. Runs application tests using:

```bash
npm test
```

2. Builds the Docker image

3. Pushes the image to GitHub Container Registry

4. Triggers deployment on Render

If the tests fail, the pipeline stops and the application is not deployed.

---

## Cloud Deployment

I deployed the application using **Render**.

I chose Render because it supports Docker deployments and provides a simple way to deploy applications without managing a virtual machine.

The deployment process is:

```
GitHub Actions
        |
        ↓
Docker Image
        |
        ↓
GitHub Container Registry
        |
        ↓
Render
```

The deployed application can be tested using:

```
GET https://<render-url>/health
```

Expected response:

```json
{
  "status": "ok"
}
```

---

## Running Locally

### Without Docker

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

---

### Running Tests

To run the automated tests:

```bash
cd app
npm test
```

The tests verify:

* Health endpoint response
* Metrics endpoint response
* Data endpoint functionality

---

## Environment Variables

I used environment variables to configure the application.

Example:

```
PORT=3000
```

A `.env.example` file is included in the repository.

Sensitive information such as tokens and credentials are stored as GitHub repository secrets and are not committed to the repository.

---

## Implementation Decisions

### Docker

I used Docker because it allows the application to run consistently in different environments.

### GitHub Actions

I used GitHub Actions to automate testing, building, and deployment instead of performing these steps manually.

### GitHub Container Registry

I used GitHub Container Registry because it integrates well with GitHub Actions and stores the Docker images used for deployment.

### Render

I chose Render as the cloud platform because it supports Docker applications and provides a suitable environment for this project without requiring paid infrastructure.

---

## Future Improvements

Possible improvements I would add:

* Add monitoring and alerts
* Add automatic rollback after failed deployments
* Add more automated tests
* Add Infrastructure as Code using Terraform
