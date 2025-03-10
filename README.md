React Application Deployment

Overview

This repository contains a React application that is built, containerized using Docker, and deployed on an AWS EC2 instance using GitHub Actions for CI/CD.

Technologies Used

React.js - Frontend framework

Node.js (Alpine) - Lightweight base image for building the app

Nginx - Web server for serving the built React application

Docker & Docker Hub - Containerization and image hosting

GitHub Actions - CI/CD pipeline for automated deployment

AWS EC2 - Cloud server for hosting the application

Directory Structure

📂 project-root
 ├── 📂 src              # React source code
 ├── 📂 public           # Public assets
 ├── 📄 Dockerfile       # Dockerfile for building the image
 ├── 📄 .github/workflows/deploy.yml  # CI/CD workflow
 ├── 📄 package.json     # Dependencies and scripts
 ├── 📄 README.md        # Documentation

Dockerfile

The Dockerfile consists of two stages:

Build Stage: Uses Node.js to build the React app.

Production Stage: Uses Nginx to serve the built app.

FROM node:alpine3.18 as build

ARG REACT_APP_NODE_ENV
ARG REACT_APP_SERVER_BASE_URL

ENV REACT_APP_NODE_ENV=$REACT_APP_NODE_ENV
ENV REACT_APP_SERVER_BASE_URL=$REACT_APP_SERVER_BASE_URL

WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

FROM nginx:1.23-alpine
WORKDIR /usr/share/nginx/html
RUN rm -rf *
COPY --from=build /app/build .
EXPOSE 80
ENTRYPOINT [ "nginx", "-g", "daemon off;" ]

CI/CD Workflow

GitHub Actions workflow automates the build and deployment process:

Workflow Triggers

Runs on every push to the main branch.

Build Job

Checks out the repository.

Logs into Docker Hub using secrets.

Builds the Docker image with appropriate build arguments.

Pushes the image to Docker Hub.

Deploy Job

Runs on a self-hosted EC2 runner.

Ensures Docker is installed and running.

Pulls the latest Docker image from Docker Hub.

Stops and removes any old running containers.

Runs the new container on port 3000.

name: Deploy React Application

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Source Code
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        run: echo "${{ secrets.DOCKER_PASS }}" | docker login -u "${{ secrets.DOCKER_NAME }}" --password-stdin

      - name: Build Docker Image
        run: docker build -t engrdawoodisrar/react-app:latest --build-arg REACT_APP_NODE_ENV='production' --build-arg REACT_APP_SERVER_BASE_URL="${{ secrets.REACT_APP_SERVER_BASE_URL }}" .

      - name: Publish Image to Docker Hub
        run: docker push engrdawoodisrar/react-app:latest

  deploy:
    needs: build
    runs-on: self-hosted
    steps:
      - name: Login to Docker Hub
        run: echo "${{ secrets.DOCKER_PASS }}" | docker login -u "${{ secrets.DOCKER_NAME }}" --password-stdin

      - name: Pull Image from Docker Hub
        run: docker pull engrdawoodisrar/react-app:latest

      - name: Stop and Remove Old Container (if exists)
        run: |
          docker stop react-app-container || true
          docker rm react-app-container || true

      - name: Run Docker Container
        run: |
          docker run -d -p 3000:80 \
            --name react-app-container \
            engrdawoodisrar/react-app:latest

Deployment on AWS EC2

Prerequisites:

AWS EC2 instance with Ubuntu installed

Docker installed on EC2 (sudo apt install docker.io -y)

Self-hosted GitHub Actions runner set up on EC2

Deployment Steps:

Clone the repository to the EC2 instance:

git clone https://github.com/your-username/your-repo.git

Run the GitHub Actions runner on EC2 to listen for deployments.

Push changes to the main branch, triggering the workflow.

The latest Docker image is pulled and deployed automatically.

Accessing the Application

Once deployed, the application will be accessible at:

http://<EC2-PUBLIC-IP>:3000

Conclusion

This setup provides a robust CI/CD pipeline for deploying a React application using Docker and GitHub Actions on AWS EC2. The entire process is automated, ensuring seamless deployment with minimal manual intervention.