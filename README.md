# Capstone Project - Containerization and Container Orchestration

## Overview
In this project, I developed a simple static website (HTML and CSS) for a company's landing page, containerized it using Docker, and deployed it on a Kubernetes cluster using **Kind** (Kubernetes in Docker). I also used **Nginx** to serve the application.

## Steps Taken

### 1. **Set Up Project Directory**

I created a new project directory for the landing page application:

    ```bash
    mkdir landing-page
    cd landing-page
    ```

#### Inside the directory, I created two files:
- index.html – Contains the HTML structure of the landing page.
- styles.css – Contains the CSS styles for the landing page.

Here’s the basic content of these files:
- index.html

    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <link rel="stylesheet" href="styles.css">
        <title>Landing Page</title>
    </head>
    <body>
        <header>
            <h1>Welcome to Our Landing Page</h1>
        </header>
        <main>
            <p>This is a simple static website containerized with Docker and deployed on Kubernetes!</p>
        </main>
        <footer>
            <p>© 2024 Company Name</p>
        </footer>
    </body>
    </html>

- styles.css

    body {
        font-family: Arial, sans-serif;
        background-color: #f4f4f4;
        margin: 0;
        padding: 0;
    }

    header {
        background-color: #1D676B;
        color: white;
        text-align: center;
        padding: 20px;
    }

    main {
        padding: 20px;
        text-align: center;
    }

    footer {
        background-color: #1D676B;
        color: white;
        text-align: center;
        padding: 10px;
        position: fixed;
        width: 100%;
        bottom: 0;
    }

### 2. Initialize Git Repository
I initialized a Git repository in the project directory to track changes:

    ```bash
        git init
    ```

### 3. Commit Initial Code
I added and committed the initial files to the Git repository:

    ```bash
    git add .
    git commit -m "Initial commit with HTML and CSS files"
    ```

### 4. Dockerize the Application
I created a Dockerfile to containerize the application using Nginx as the base image. The Dockerfile copies the HTML and CSS files into the appropriate Nginx directory:

- Dockerfile

    # Use the official Nginx image from Docker Hub
    FROM nginx:alpine

    # Copy the HTML and CSS files into the container
    COPY index.html /usr/share/nginx/html/
    COPY styles.css /usr/share/nginx/html/

    # Expose port 80
    EXPOSE 80

### 5. Push Docker Image to Docker Hub
After building the Docker image, I logged into Docker Hub and pushed the image:

    ```bash
    docker build -t your-dockerhub-username/landing-page .
    docker login
    docker push your-dockerhub-username/landing-page
    ```

### 6. Set Up Kind Kubernetes Cluster
To simulate a Kubernetes cluster locally, I installed and set up Kind (Kubernetes in Docker) and created a new cluster using a configuration file.

- kind-config.yaml

    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    nodes:
    - role: control-plane
    - role: worker

- I created the cluster with:

    ```bash
    kind create cluster --config kind-config.yaml
    ``` 

### 7. Deploy the Application to Kubernetes
I created a Deployment YAML file for the landing page application, specifying the Docker image I pushed earlier:

- deployment.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: landing-page-deployment
    spec:
    replicas: 2
    selector:
        matchLabels:
        app: landing-page
    template:
        metadata:
        labels:
            app: landing-page
        spec:
        containers:
            - name: landing-page
            image: your-dockerhub-username/landing-page
            ports:
                - containerPort: 80

- I applied the deployment to the cluster:

    ```bash
    kubectl apply -f deployment.yaml
    ```

### 8. Create a Service (ClusterIP)
I created a ClusterIP Service YAML to expose the application within the Kubernetes cluster:

- service.yaml

    apiVersion: v1
    kind: Service
    metadata:
    name: landing-page-service
    spec:
    selector:
        app: landing-page
    ports:
        - protocol: TCP
        port: 80
        targetPort: 80
    type: ClusterIP

- I applied the service to the cluster:

    ```bash
    kubectl apply -f service.yaml
    ```

### 9. Access the Application
To access the application, I used kubectl port-forward to forward port 8080 on my host machine to port 80 in the Kubernetes service:

    ```bash
    kubectl port-forward service/landing-page-service 8080:80
    ```

Then, I accessed the application on http://localhost:8080 from my browser.

## Issues Encountered & Resolutions
### 1. Issue: "ImagePullBackOff" Error
I encountered an ImagePullBackOff error when deploying the pod. This was due to an issue with pulling the Docker image from Docker Hub.

Solution: I verified the image name and ensured I was logged into Docker Hub. I also checked the image's availability on Docker Hub by pulling it manually on my VM.

### 2. Issue: Unable to Access the Application
Initially, I was unable to access the application on my browser because I was using a VM in Oracle VirtualBox. Since the VM was using NAT mode, it isolated the network, and I couldn't directly access the application from the host machine.

Solution: I tried to configure port forwarding in VirtualBox and set up a forwarding rule to allow traffic from my host machine to the VM on port 8080. Alternatively, I could have switched the VM’s network mode to Bridged Adapter to make it directly accessible from the host.

### 3. Issue: Application Not Loading in Browser
Even though the port forwarding was successful, the application wasn't loading in the browser.

Solution: I verified that the Nginx server inside the container was running correctly by using kubectl exec to curl the service. The HTML response showed that everything was working within the container. I also checked the service and deployment YAML files for correct configurations and confirmed that the application was correctly exposed via port 8080.

## Conclusion
This project demonstrated the complete process of containerizing a simple static website using Docker and deploying it to a Kubernetes cluster with Kind. I also learned about the common issues faced while working with Kubernetes in a virtualized environment and how to resolve them.

With everything set up and running, I was able to successfully deploy the landing page application and access it through port forwarding.
