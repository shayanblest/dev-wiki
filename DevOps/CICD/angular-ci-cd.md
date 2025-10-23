# 🚀 CI/CD for Angular using GitHub Actions

This document explains how to set up a **CI/CD pipeline for Angular applications** using **GitHub Actions**.

---

## 🧩 Goals

- Automatically run tests on every Pull Request  
- Build the project for production on every push to `main`  
- Automatically deploy the latest build to the server  

---

## ⚙️ Pipeline Overview

1. **Continuous Integration (CI)**
   - Install dependencies (`npm ci`)
   - Run unit tests (`npm run test`)
   - Build the Angular app (`ng build --configuration production`)

2. **Continuous Deployment (CD)**
   - Upload the built files (`dist/`) to the server
   - Restart the service or reload Nginx on the server

---

## 🧱 GitHub Actions Workflow File

Create the following file in your repository:  
`.github/workflows/angular-ci-cd.yml`

```yaml
name: Deploy Frontend to Dev Server

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Choose environment"
        required: true
        type: choice
        options:
          - staging
          - production

env:
  DOCKER_REGISTRY: docker.buludweb.ir
  IMAGE_NAME: <registery-name>/front

jobs:
  build:
    name: Build Docker Image
    runs-on: ${{ github.event.inputs.environment }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Login to Docker Registry
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login ${{ env.DOCKER_REGISTRY }} -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

      - name: Build Docker Image
        run: |
          docker build --build-arg ENV=${{ github.event.inputs.environment }} \
            -t ${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} .
          docker push ${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}

  deploy:
    name: Deploy Docker Container
    needs: build
    runs-on: ${{ github.event.inputs.environment }}
    steps:
      - name: Login to Docker Registry
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login ${{ env.DOCKER_REGISTRY }} -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

      - name: Deploy Frontend
        run: |
          CONTAINER_NAME=<container-name>
          IMAGE_TAG=${{ github.sha }}

          echo "Stopping old container..."
          docker stop $CONTAINER_NAME || true
          docker rm $CONTAINER_NAME || true

          echo "Starting new container..."
          docker run -d \
            --name $CONTAINER_NAME \
            --restart always \
            -p 4200:80 \
            ${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}:$IMAGE_TAG

          echo "✅ Frontend deployed on ${{ github.event.inputs.environment }}"
