
# DevOps, Docker, Kubernet, ingress, reddit-webapp and aws project

set up your Kubernetes project with two instances, one for the CI server and another for the deployment server:

1. **CI Server (t2.micro):**
   - Launch an EC2 instance with the Ubuntu AMI (t2.micro).
   - SSH into the instance.
   - Update the package list and install Docker:
     ```bash
     sudo apt update
     sudo apt install docker.io -y
     ```
   - Add your user to the docker group to run Docker commands without sudo:
     ```bash
     sudo usermod -aG docker $USER && newgrp docker
     ```
   - Clone your GitHub repository for the project:
     ```bash
     git clone https://github.com/vishal815/DevOps-Dcoker-Kubernet-reddit-webapp-project.git
     ```

2. **Deployment Server (t2.medium):**
   - Launch another EC2 instance with the Ubuntu AMI (t2.medium).
   - SSH into the instance.
   - Install Docker, Minikube, and kubectl:
     ```bash
     sudo apt update
     sudo apt install docker.io -y
     sudo usermod -aG docker $USER && newgrp docker

     curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
     sudo install minikube-linux-amd64 /usr/local/bin/minikube 

     sudo snap install kubectl --classic
     minikube start --driver=docker
     ```
  - Note:- Make sure you open the 3000 port in a security group of your Ec2 Instance.
    ![image](https://github.com/vishal815/DevOps-Dcoker-Kubernet-reddit-webapp-project/assets/83393190/359cb70e-2428-4ee8-b1a6-d463297d6d71)



These steps will set up your CI server with Docker installed and your GitHub repository cloned, and your deployment server with Docker, Minikube, and kubectl installed.




# Here are the steps you can follow on your CI server (t2.micro) after installing Docker:
- replace vishallazrus with your dockerhub user name.

Step 1: Clone the Source Code
SSH into your CI server instance and clone the source code from your GitHub repository:
```bash
git clone https://github.com/vishal815/DevOps-Dcoker-Kubernet-reddit-webapp-project.git
```

Step 2: Containerize the Application using Docker
Create a Dockerfile in the root of your project directory with the following content:
```Dockerfile
FROM node:14-alpine

WORKDIR /app

COPY . .

RUN npm install

EXPOSE 3000

CMD ["npm", "start"]
```

Step 3: Building Docker Image
Build the Docker image using the Dockerfile you created:
```bash
docker build -t vishallazrus/reddit-webapp .
```

Step 4: Push the Image to DockerHub
Login to DockerHub:
```bash
docker login
```
Push the Docker image to DockerHub:
```bash
docker push vishallazrus/reddit-webapp
```

After pushing the image to DockerHub, you can verify its presence by logging into hub.docker.com and checking if the image is listed there.
![image](https://github.com/vishal815/DevOps-Dcoker-Kubernet-reddit-webapp-project/assets/83393190/1780d5e6-6ac0-4412-aaf7-15b5cf0a6dbd)

# -  -  -   -   -   -   -  -  -  -  -  -  -  -  -  -   -  -  -  -  -  -  -  -  -   -  -  

# Naw next stape on Deployment server 

Here's a step-by-step guide with the correct commands to follow on your Deployment server after installing Docker and Minikube:

1. Create a directory and navigate to it:
```bash
mkdir k8s
cd k8s
```

2. Write a Kubernetes Manifest File for Deployment:
```bash
nano deployment.yaml
```
Paste the following content into the file and save it:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reddit-webapp-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: reddit-webapp
  template:
    metadata:
      labels:
        app: reddit-webapp
    spec:
      containers:
      - name: reddit-webapp
        image: vishallazrus/reddit-webapp
        ports:
        - containerPort: 3000
```

3. Write a Kubernetes Manifest File for Service:
```bash
nano service.yaml
```
Paste the following content into the file and save it:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: reddit-webapp-service
spec:
  type: NodePort
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30000
  selector:
    app: reddit-webapp
```

4. Deploy the App to Kubernetes & Create a Service:
Apply the deployment and service YAML files to your Kubernetes cluster:
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

If You want to check your deployment & Service use the command ```kubectl get deployment``` & ```kubectl get services```
![Screenshot (680)](https://github.com/vishal815/DevOps-Dcoker-Kubernet-reddit-webapp-project/assets/83393190/d7053338-3de9-4a8d-a715-e84106feaf2f)


5. Access the Application:
To access the application, you can use Minikube's service command:
```bash
minikube service reddit-webapp-service
```
This command will open the application in your default web browser.

Alternatively, you can use port forwarding to access the application:
```bash
kubectl port-forward svc/reddit-webapp-service 3000:3000 --address 0.0.0.0 &
```
Now you can access the application using the Deployment server's IP address and port 3000, like so:
```
http://<ip_address_of_deployment_server>:3000
```

![Screenshot (681)](https://github.com/vishal815/DevOps-Dcoker-Kubernet-reddit-webapp-project/assets/83393190/9474b7e3-0b45-40b9-8f2f-04d2a27127ce)

# -  -  -   -   -   -   -  -  -  -  -  -  -  -  -  -   -  -  -  -  -  -  -  -  -   -  -  
# 😃 We finished with project deployment for the Configure Ingress part.

Here are the correct steps and commands to configure Ingress and expose your application in Kubernetes project:

Step 6: Configure Ingress

1. Create an ingress.yaml file:
```bash
nano ingress.yaml
```

2. Insert the following code into the file and save it (replace "domain.com" and "reddit-clone-service" with your actual domain and service names):
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-reddit-app
spec:
  rules:
  - host: "domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000
  - host: "*.domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000
```

3. Enable Ingress in Minikube:
```bash
minikube addons enable ingress
```

4. Apply the Ingress configuration:
```bash
kubectl apply -f ingress.yaml
```

5. Verify that the Ingress resource is running correctly:
```bash
kubectl get ingress ingress-reddit-app
```

Step 8: Expose the App

1. Expose your deployment:
```bash
kubectl expose deployment reddit-clone-deployment --type=NodePort
```

2. Test your deployment using curl:
```bash
curl -L http://<minikube_ip>:<node_port>
```
Replace `<minikube_ip>` and `<node_port>` with the actual values from your setup.

3. Expose your app service using port forwarding:
```bash
kubectl port-forward svc/reddit-clone-service 3000:3000 --address 0.0.0.0 &
```

4. Test Ingress:
Use curl to test your Ingress configuration:
```bash
curl -L http://domain.com/test
```
Replace "domain.com/test" with the actual URL path specified in your Ingress configuration.

Additionally, you can access the deployed application on your EC2 instance using its IP address and port 3000, but make sure to open port 3000 in the security group of your EC2 instance.

# -  -  -   -   -   -   -  -  -  -  -  -  -  -  -  -   -  -  -  -  -  -  -  -  -   -  -  
# If you are stoping your EC2 instance make sure next time start kubelet 
If you are stopping your instance and want to access your application using `http://<ip_address_of_deployment_server>:3000`, you will need to follow these steps:


1. Start the Kubernetes service and make sure your application's pods are running:
```bash
sudo systemctl start kubelet
kubectl get pods
```
Ensure that your security group allows inbound traffic on port 3000 to access your application externally.
Note:- The shown IP is my temporary IP it will be destroyed after my instance restart.
# Thank You Happy learning😃
