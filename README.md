
# Dockerizing and Deploying a MERN App with Kubernetes

I found a full-stack MERN application that manages the basic information of employees. The app uses an employee database from the MongoDB Atlas database and then display it using a React. This is the link to the original directory: [doananhtingithub40102- mern-app](https://github.com/doananhtingithub40102/mern-app)

What I did was dockerize and deploy this MERN app using Kubernetes. Below is a **step-by-step, super-detailed guide** that covers everything:
---

## **1. Prerequisites**
Download all the prerequisites from the requirements file. Each individual installation and setup takes no more than 5 minutes. I had the entire setup ready to go in 30 minutes from scratch.
---

## **2. Dockerizing the MERN App**

### **2.1 Folder Structure**

First thing I did was put a **Dockerfile** in the root of both the `client` and `server` directories.

We need separate Dockerfiles for the server and client components of the MERN application because they are distinct applications with different purposes, dependencies, and runtime requirements.
As you can see that I have made a separate dockerfile for server and client.

---

### **2.2 Create a Docker Compose File**

#### Why Create a `docker-compose.yml` File?

A MERN application typically involves multiple containers:
- A **MongoDB database** container.
- A **server** (backend) container.
- A **client** (frontend) container.

Instead of running `docker run` commands for each service, `docker-compose` orchestrates the startup, networking, and configuration of all these containers in a single file.

In a MERN stack, the **server** depends on the **database** being available before it starts. `docker-compose` manages these dependencies using `depends_on`, ensuring proper startup order.

`docker-compose` automatically creates a **bridge network** so all containers can communicate using service names (e.g., `mongo`, `server`, `client`).

#### Why Place `docker-compose.yml` in the Project Root?

The project root is where all components (**client**, **server**, and **database**) are defined. Placing `docker-compose.yml` here allows it to access all necessary resources.

It’s a convention in most projects to have `docker-compose.yml` at the root for easy discovery and execution.

---

## **3. Testing Locally with Docker**
1. Build and start the containers:
   ```bash
   docker-compose up --build
   ```
2. Open the app in the browser:
   - Client: `http://localhost:3000`
   - Server: `http://localhost:5000`

---

## **4. Deploying to Kubernetes**

### **4.1 Create Kubernetes Manifests**

#### Why Create Separate Deployments and Services for Each Kubernetes Component?

In Kubernetes, each component (e.g., MongoDB, server, client) requires its own **Deployment** and **Service** to ensure clear separation of concerns, scalability, and independent management.

**1. Separate Deployments**
A **Deployment** manages the desired state of an application, ensuring that the specified number of pod replicas are running at all times.  
Each component (MongoDB, server, client) performs a unique role, so they need individual Deployments to:
- Define their container image and environment.
- Enable independent scaling (e.g., scale backend pods without affecting MongoDB).

**2. Separate Services**
A **Service** provides a stable network interface to access pods, even if their underlying IPs change.  
Each component needs its own Service to:
- Expose it to other components or external users (e.g., MongoDB for server, client for end-users).
- Decouple internal logic (e.g., backend APIs communicate with the database via its service).

### What is a Deployment?

A **Deployment** is a Kubernetes object that:
- Defines how to run and manage pods for an application.
- Ensures desired state (e.g., number of replicas, container image version).
- Provides features like rolling updates and self-healing.

### What is a Service?

A **Service** is a Kubernetes object that:
- Exposes pods to the network (internally or externally).
- Provides a consistent endpoint using DNS names, even if pod IPs change.
- Types of Services:
  - **ClusterIP**: Internal communication within the cluster.
  - **NodePort**: Exposes the application on a port of each node.
  - **LoadBalancer**: Distributes traffic using a cloud provider's load balancer.

You can also see the k8s folder I made and all the respective yaml files.

---

### **4.2 Apply Kubernetes Manifests**
#### **Step 1: Ensure Docker is Running**

Before applying the Kubernetes manifests, make sure Docker is running, as Kubernetes will use Docker containers for the pods.

1. **Start Docker**: Ensure Docker is installed and running on your local machine.
   - If using Docker Desktop, start it from the application.
   - If using Docker CLI, ensure it is running by checking the status:
     ```bash
     docker info
     ```

2. **Check Docker Images**: Ensure that the images for the components (frontend, backend, database) are built and available in Docker. If not, build them first:
   - Navigate to the `client` and `server` directories and build Docker images.
   - For example:
     ```bash
     docker build -t my-client ./client
     docker build -t my-server ./server
     ```

#### **Step 2: Start a Local Kubernetes Cluster Using Minikube**

1. **Start Minikube**:
   - Minikube creates a local Kubernetes cluster on your machine.
   ```bash
   minikube start
   ```
   This command will download the necessary Kubernetes components and set up a local Kubernetes environment. The process might take a few minutes depending on your machine's resources.

2. **Verify Minikube is Running**:
   - Once Minikube has started successfully, verify the Kubernetes cluster is up and running:
   ```bash
   kubectl cluster-info
   ```


#### **Step 3: Applying Everything**

1. **Navigate to the `k8s/` Directory**:
   - Your Kubernetes manifests (such as `mongo.yaml`, `server.yaml`, `client.yaml`) should be stored in the `k8s/` directory within your project.
   - Use the `cd` command to change to this directory:
     ```bash
     cd mern-app-main/k8s
     ```

2. **Apply Kubernetes Manifests**:
   - The `kubectl apply` command will create the resources (Deployments, Services, etc.) as defined in the Kubernetes manifests.
   ```bash
   kubectl apply -f .
   ```
   - This command will apply all the YAML files within the `k8s/` directory. If you want to apply a specific manifest file, you can run:
     ```bash
     kubectl apply -f client.yaml
     kubectl apply -f server.yaml
     kubectl apply -f mongo.yaml
     ```

3. **Verify Resources are Created**:
   - After applying the manifests, check if the deployments and services have been successfully created:
   ```bash
   kubectl get deployments
   kubectl get services
   ```
   This will show the status of your deployments and services, ensuring everything is up and running.


#### **Step 4: Access the Client Using the LoadBalancer IP**

1. **Get the External IP**:
   - If your service is of type `LoadBalancer`, Minikube will provide a local IP to access the client. Use the following command to get the LoadBalancer's IP:
   ```bash
   minikube service client --url
   ```

2. **Access the Client**:
   - After running the command above, you’ll receive an URL (e.g., `http://192.168.99.100:30001`). Open this URL in your browser to access the client (frontend).

3. **Verify Communication**:
   - Check if the frontend is communicating correctly with the backend and the MongoDB database.


## **5. Developer Tips and Common Issues**

### **5.1 Common Tips**
- Always use environment variables to store sensitive data like database URIs.
- Test each component (client, server, and database) individually in Docker before integrating them into Kubernetes.

### **5.2 Common Issues**
1. **Docker build errors**:
   - Ensure `package.json` is in the same directory as the `Dockerfile`.
2. **Kubernetes connectivity issues**:
   - Check pod logs:
     ```bash
     kubectl logs <pod-name>
     ```

This guide should help you take your simple MERN app and turn it into a Dockerized, Kubernetes-ready application! 

Musfirah, signing out! 🙌
