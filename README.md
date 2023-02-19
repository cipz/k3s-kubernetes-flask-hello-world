# Hello World from Flask in a k3s deployed container

This is yet another simple tutorial on how to deploy a Flask application on Kubernetes, specifically on k3s.

## Flask Hello World

The python code that composes the Python application is stored in the `app.py` file and is as follows: 

    from flask import Flask

    app = Flask(__name__)

    @app.route("/")
    def hello_world():
        return "<p>This is a Hello World application</p>"

    if __name__ == "__main__":
        app.run(host='0.0.0.0', debug=True)

This can be launched with the command  `python app.py` and will start a service reachable at `http://localhost:5000`.

The only requirement for this to run is [Flask](https://pypi.org/project/Flask/).

## Docker

Inside the Dockerfile, there is the specification to create the image that will be later deployed in the k3s cluster.

The Dockerfile is as follows:

    FROM python:3.10-alpine
    WORKDIR /app
    COPY . .
    RUN pip install -r requirements.txt
    EXPOSE 5000
    CMD ["python", "app.py"]

Build the image with: `docker build -t k3s-kubernetes-flask-hello-world .`
    
To run the image use `docker run k3s-kubernetes-flask-hello-world -p 5000:5000`.
The service will again be reachable at `http://localhost:5000`.

## Deployment configuration

Now that the image has been built, it is time to configure the k3s deployment as follows:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: flask-app-deployment
    spec:
    replicas: 3
    selector:
        matchLabels:
        app: flask-app-pod
    template:
        metadata:
        labels:
            app: flask-app-pod
        spec:
        containers:
        - name: flask-app-pod-image
            image: kubernetes-flask-hello-world
            imagePullPolicy: Never
            resources: 
            limits:
                memory: "128Mi"
                cpu: "500m"
            ports:
            - containerPort: 5000

    ---
    apiVersion: v1
    kind: Service
    metadata:
    name: flask-app-service
    spec:
    selector:
        app: flask-app-pod
    ports:
    - protocol: "TCP"
        port: 6000
        targetPort: 5000
    type: LoadBalancer

## Importing the image in k3s

Save the image as into a `.tar` archive and import it in k3s via `containerd`:

    docker save --output k3s-kubernetes-flask-hello-world.tar k3s-kubernetes-flask-hello-world
    sudo k3s ctr images import kubernetes-flask-hello-world.tar

Another option that does it in one line and does not require saving the image:

    docker save k3s-kubernetes-flask-hello-world | sudo k3s ctr images import -

## Connecting to the pods

## References
- [Import images to k3s without docker registry](https://cwienczek.com/2020/06/import-images-to-k3s-without-docker-registry/)
