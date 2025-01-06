# BASIC FRONTEND APPLICATION WITH DOCKER AND KUBERNETES

The goal of this project is to containerize a simple static website for a company's landing page using Docker, deploy it to a Kubernetes cluster, and access it through Nginx.

This documentation will seek to highlight steps I took to accomplish the goal of this project and will be divided into various tasks.

## TASK 1 - SET UP YOUR PROJECT

The first thing I did was to create a new project directory. I named it "k8s-project".

I then proceeded to create two files:

- index.html
- styles.css

![dirfiles](./img/1%20mkdir.jpg)
![webfiles](./img/2%20create-web-app.jpg)

## TASK 2 - INITIALIZE AND COMMIT GIT

Next, I want to store, change, merge and collaborate on files. As such, I aim to initialize Git on this directory. To initialize git, I did the following:

In this case, I did not need to configure Git using git config because git is already configured.

- Created a repository on Github named "k8s-project".
- Ran the following commands on my cli:
  - echo "# k8s-project" >> README.md
  - git init
  - git add README.md
  - git commit -m "first commit"
  - git branch -M main
  - git remote add origin <https://github.com/madusug/k8s-project.git>
  - git push -u origin main
- Added the index.html and styles.css to the staging area using "git add ."
- Committed index.html and styles.css and pushed using "git commit -m "commit web files"" and "git push -u origin main"

![git](./img/3%20git.jpg)
![git1](./img/3%20git1.jpg)

## TASK 3 - DOCKERIZE THE APPLICATION

In order for me to have a streamlined development process, I decided to use Docker to help maintain a consistent environment. This environment will be maintained across various stages of development, reducing the time spent on fixing environment-related issues.

Using Nginx as the base image, I proceeded to create a Dockerfile wherein I copied HTML and CSS files into the Nginx HTML directory. See below:

```dockerfile
# Use the official NGINX base image
FROM nginx:latest

# Set the working directory in the container
WORKDIR  /usr/share/nginx/html/

# Copy the local HTML file to the NGINX default public directory
COPY index.html styles.css usr/share/nginx/html/

# Expose port 80 to allow external access
EXPOSE 80

# No need for CMD as NGINX image comes with a default CMD to start the server
```

The next step here is to build the image. I did this using: "docker build -t new-app ."
I then proceeded to login to docker hub while doing the following:

- Login to docker on my cli using: docker login
- Tag the image: docker tag 01fd3bb5c3ff distinctugo/new-app:v1
- Push image to docker: docker push distinctugo/new-app

![dock-push](./img/5%20dock-push.jpg)

## TASK 4 - SET UP A KIND KUBERNETES CLUSTER

Kind, short for Kubernetes IN Docker, is a tool that allows users to create kubernetes clusters on their local machines for testing and development.

To install kind, I ran the following command:

```
choco install kind
```

At first I did not know much about kind but after research, I found out that the first step is to create a configuration file for the kind cluster. The kind configuration file is used to create and configure a kubernetes cluster using the open source tool, Kind.

### I created the kind configuration file called, kind-cluster.yaml with the following as input:

```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
```
![kind-config](./img/6%20kind-config.jpg)


### To create the cluster, I ran the following command:

```
kind create cluster --name ugo-kind-cluster --config kind-cluster.yaml
```

![kind-create](./img/7%20kind-create.jpg)

## TASK 5 - DEPLOY TO KUBERNETES

The next step is to create a kubernetes deployment yaml file where I will specify the image to be used, desired replicas and well as pod configuration, container specifications, readiness and liveness probes, resource requests and limits and so on.

### For this I created a file named app.yaml with the following code

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
        - name: app-nginx-container
          image: distinctugo/new-app:v1
          ports:
            - containerPort: 80
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 15
            periodSeconds: 20
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "128Mi"
              cpu: "500m"

```

### Apply Deployment to Cluster

```
kubectl apply -f app.yaml
```

![deploy](./img/8%20deploy.jpg)

## TASK 6 - CREATE A SERVICE (CLUSTERIP)

The next step for me was to create a kubernetes service yaml file specifying the type as ClusterIP. ClusterIP exposes the service on a cluster internal IP. Choosing clusterip makes the service only reachable from within the cluster. Since I don't want to expose the service to the public internet, this is a viable choice.

### To do this, I created a file called app-services.yaml with the following yaml syntax:

```
apiVersion: v1
kind: Service
metadata:
  name: james-nginx-svc
  labels:
    app: nginx
spec:
  selector:
    app: nginx-pod
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

### Apply Service

```
kubectl apply -f app-services.yaml
```

## TASK 7 - ACCESS THE APPLICATION

For this step, I ran into a big issue trying to access the application. I tried various ways I had learned previously using minikube to access the application locally but nothing seemed to work. After much research, I found that what I needed to do was port-forwarding specifically and the ports I would need are 8080:80. After figuring this out, I was able to access the application by doing the following:

- run, kubectl get services, to see the name of the service since I would use it in my post-forward syntax

```
kubectl port-forward service/james-nginx-svc 8080:80
```

![port](./img/9%20port.jpg)

This ran successfully. I was able to see my application locally using:

```
localhost:8080
```

![local-web](./img/10%20local-web.jpg)
