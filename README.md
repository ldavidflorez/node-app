# Node App

A simple Node.js application that runs an HTTP server, designed for containerized deployment in Kubernetes.

## Description

This application serves a basic "Hello, Kubernetes with Kustomize!" message on port 3000. It's containerized using Docker and includes a CI/CD pipeline for building and pushing images to Oracle Cloud Infrastructure Registry (OCIR), followed by triggering deployments via ArgoCD.

## Prerequisites

- Node.js (for local development)
- Docker (for building images)
- Access to Oracle Cloud Infrastructure (OCI) with OCIR
- GitHub repository with Actions enabled

## Configuration

To use the CI/CD pipeline, set up the following secrets in your GitHub repository settings:

- `OCI_DOCKER_REPO`: The full OCIR repository name (e.g., `us-ashburn-1.ocir.io/tenancy/repository-name`).
- `OCI_COMPARTMENT_OCID`: The OCID of the OCI compartment containing the repository.
- `OCI_AUTH_TOKEN`: An OCI authentication token for accessing OCIR.
- `PAT`: A GitHub Personal Access Token with `repo` scope, used to dispatch events to the ArgoCD configuration repository.
- `OCI_CLI_USER`: The OCID of the OCI user.
- `OCI_CLI_TENANCY`: The OCID of the OCI tenancy.
- `OCI_CLI_FINGERPRINT`: The fingerprint of the OCI API key.
- `OCI_CLI_KEY_CONTENT`: The content of the OCI private key (in PEM format).
- `OCI_CLI_REGION`: The OCI region (e.g., `us-ashburn-1`).

### Creating a GitHub Personal Access Token (PAT)

1. Go to GitHub and log in to your account.
2. Click on your profile picture in the top right corner and select **Settings**.
3. In the left sidebar, scroll down and click **Developer settings**.
4. Click **Personal access tokens** > **Tokens (classic)**.
5. Click **Generate new token (classic)**.
6. Give it a descriptive name, e.g., "ArgoCD Dispatch Token".
7. Select the scopes: Check `repo` (full control of private repositories) to allow dispatching events.
8. Click **Generate token**.
9. Copy the token immediately (you won't see it again).
10. Add this token as the `PAT` secret in your repository settings under **Secrets and variables** > **Actions**.

## CI/CD Workflow

The GitHub Actions workflow (`.github/workflows/CI.yml`) triggers on pushes to the `main` branch:

1. Checks out the code.
2. Authenticates with OCIR using the provided secrets.
3. Builds the Docker image.
4. Pushes the image to OCIR with a tag based on the commit SHA.
5. Dispatches a repository event to the ArgoCD config repository (`${{ github.actor }}/argocd-node-app-config`) to trigger deployment.

### Testing the CI Pipeline

To test the CI pipeline:

1. Ensure all required secrets are configured in your repository settings.
2. Make a small change to the code (e.g., update the message in `app.js`).
3. Commit and push the changes to the `main` branch:
   ```bash
   git add .
   git commit -m "Test CI pipeline"
   git push origin main
   ```
4. Go to the **Actions** tab in your GitHub repository.
5. You should see a new workflow run for the CI job.
6. Monitor the logs to ensure the build, push, and dispatch steps complete successfully.
7. Check the ArgoCD configuration repository for the dispatched event if applicable.

If the workflow fails, review the error logs and verify your secrets and OCI setup.

## Local Development

### Running with Node.js

1. Install dependencies:
   ```bash
   npm install
   ```

2. Run the application:
   ```bash
   node app.js
   ```

3. Access the app at `http://localhost:3000`.

### Running with Docker

1. Build the image:
   ```bash
   docker build -t node-app .
   ```

2. Run the container:
   ```bash
   docker run -p 3000:3000 node-app
   ```

3. Access the app at `http://localhost:3000`.

## Deployment

The application is designed to be deployed to Kubernetes using ArgoCD. The CI pipeline triggers updates in the ArgoCD configuration repository, which should handle the deployment manifests.

Ensure your ArgoCD setup is configured to pull the latest image from OCIR based on the dispatched event.