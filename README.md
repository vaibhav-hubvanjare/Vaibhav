apiVersion: apps/v1
kind: Deployment
metadata:
  name: wisecow-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wisecow
  template:
    metadata:
      labels:
        app: wisecow
    spec:
      containers:
      - name: wisecow
        image: <your_registry>/wisecow:latest
        ports:
        - containerPort: 8000
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and Push Docker Image
        run: |
          docker build -t <your_registry>/wisecow:latest .
          docker push <your_registry>/wisecow:latest

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: Set up kubectl
        uses: azure/setup-kubectl@v3
        with:
          version: 'latest'

      - name: Deploy to Kubernetes
        env:
          KUBECONFIG: ${{ secrets.KUBECONFIG }}
        run: |
          kubectl set image deployment/wisecow-deployment wisecow=<your_registry>/wisecow:latest
