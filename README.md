# HelloWorldApp - .NET Core Web Application
This project demonstrates how to build, Dockerize, and deploy a simple .NET Core web application to a Kubernetes cluster using Minikube. The application is exposed using a Kubernetes Service and supports scaling and health checks through liveness and readiness probes.

# Table of Contents
1. Prerequisites
2. Project Setup
3. Dockerize the Application
4. Setup Kubernetes with Minikube
5. Deploy to Kubernetes
6. Accessing the Application
7. Scaling the Application
8. Health Checks - Liveness and Readiness Probes
9. Bonus: Cleanup Resources
    
# Prerequisites
# Before getting started, make sure you have the following installed:

1. .NET Core SDK
2. Docker
3. Minikube
4. kubectl
5. (Optional) Git for version control

# Project Setup

# Create a new .NET Core Web App:
dotnet new web -o HelloWorldApp
cd HelloWorldApp

# Run the application locally (optional):
dotnet run
The application should now be available at http://localhost:5000.

# Dockerize the Application
# Create a Dockerfile in the root of your project:
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base<br>
WORKDIR /app<br>
EXPOSE 80<br>

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build<br>
WORKDIR /src<br>
COPY . .<br>
RUN dotnet restore<br>
RUN dotnet publish -c Release -o /app<br>

FROM base AS final<br>
WORKDIR /app<br>
COPY --from=build /app .<br>
ENTRYPOINT ["dotnet", "HelloWorldApp.dll"]<br>

# Build the Docker Image:
docker build -t helloworldapp:1.0 .<br>
Setup Kubernetes with Minikube<br>

# Start Minikube:
minikube start

# Use Minikube's Docker environment:
eval $(minikube docker-env)

# Build the Docker image inside Minikube:
docker build -t helloworldapp:1.0 .

# Deploy to Kubernetes
# Create a Kubernetes Deployment file deployment.yaml:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: helloworld
        image: helloworldapp:1.0
        ports:
        - containerPort: 80
        
# Create a Kubernetes Service file service.yaml:
apiVersion: v1
kind: Service
metadata:
  name: helloworld-service
spec:
  type: NodePort
  selector:
    app: helloworld
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30001  # NodePort range is typically 30000-32767
# Apply the Deployment and Service configurations:

kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Verify the Pods:
kubectl get pods

# Verify the Service:
kubectl get services

# Accessing the Application
# Access the Application using Minikube:
minikube service helloworld-service

# This will open a browser or provide a URL to access the application.

# Alternatively, you can access the application by visiting:
http://<minikube-ip>:30001

# Scaling the Application
To scale the deployment (e.g., to 5 replicas):

# Scale the deployment:
kubectl scale deployment helloworld-deployment --replicas=5

# Verify the scaling:
kubectl get pods

# Health Checks - Liveness and Readiness Probes
# Add Health Check Endpoint in the .NET application (in Startup.cs):
app.MapGet("/health", () => "Healthy");

# Modify the deployment.yaml to include probes:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: helloworld
        image: helloworldapp:1.0
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
          
# Reapply the deployment configuration:
kubectl apply -f deployment.yaml

# Bonus: Cleanup Resources
To clean up all resources from your Minikube cluster:

# Delete the service:
kubectl delete service helloworld-service

# Delete the deployment:
kubectl delete deployment helloworld-deployment

# Stop Minikube (optional):
minikube stop

# Conclusion
This project demonstrates a simple example of Dockerizing a .NET Core web application and deploying it to a Kubernetes cluster using Minikube. We also covered scaling and health checks through liveness and readiness probes to ensure the application's robustness.
