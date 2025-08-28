# Audio Wave: A Microservices-Based Video-to-MP3 Converter

![Python](https://img.shields.io/badge/Python-3.x-blue) ![Flask](https://img.shields.io/badge/Flask-black) ![Docker](https://img.shields.io/badge/Docker-blue) ![Kubernetes](https://img.shields.io/badge/Kubernetes-blue) ![RabbitMQ](https://img.shields.io/badge/RabbitMQ-orange) ![MongoDB](https://img.shields.io/badge/MongoDB-green)

Audio Wave is a distributed, cloud-native application that converts uploaded video files into MP3 audio format. The project is built entirely on a microservices architecture, demonstrating best practices for asynchronous communication, containerization, and orchestration with Kubernetes.

## Architecture

The system is composed of four main services that communicate asynchronously through a message queue. An API Gateway acts as the single entry point for all client requests.


* **API Gateway**: The client-facing service that handles all incoming requests, including authentication, file uploads, and downloads.
* **Auth Service**: Manages user authentication using a MySQL database and issues JWTs for securing API endpoints.
* **Converter Service**: A consumer service that listens for new video uploads via RabbitMQ, performs the video-to-MP3 conversion, and stores the result in MongoDB.
* **Notification Service**: A consumer service that listens for completed conversion events and notifies the user via email.

## Key Features

* **Asynchronous Processing**: Uses **RabbitMQ** as a message broker to decouple services, ensuring high throughput and resilience. If a downstream service is slow or offline, tasks remain in the queue to be processed later.
* **JWT-Based Security**: The system is secured using JSON Web Tokens. The Auth service issues tokens upon successful login, which are then validated by the API Gateway for all subsequent requests.
* **Large File Handling**: Leverages **MongoDB GridFS** to efficiently store and stream video and audio files that exceed MongoDB's 16MB document size limit.
* **Containerization & Orchestration**: Each microservice is containerized using **Docker**. The entire application stack is deployed and managed on a local **Kubernetes** cluster (Minikube), automating deployment, scaling, and networking.
* **Scalable by Design**: The use of competing consumers on the RabbitMQ queues allows the `converter` and `notification` services to be scaled horizontally to handle increased load.

## Technology Stack

* **Backend:** Python 3, Flask
* **Databases:** MongoDB (for file storage), MySQL (for user authentication)
* **Messaging:** RabbitMQ
* **DevOps:** Docker, Kubernetes (Minikube), K9s
* **Authentication:** PyJWT (JSON Web Tokens)
* **Media Processing:** MoviePy

## Getting Started

### Prerequisites
* [Docker Desktop](https://www.docker.com/products/docker-desktop/)
* [Minikube](https://minikube.sigs.k8s.io/docs/start/)
* [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* Python 3

### Running the Application

1.  **Start Kubernetes Cluster:**
    ```bash
    minikube start
    ```

2.  **Build and Push Docker Images:**
    Navigate to each service directory (`auth`, `gateway`, `converter`, `notification`) and build the Docker image. You will need to tag and push these images to your own Docker Hub repository.
    ```bash
    # Example for the auth service
    cd auth
    docker build -t your_own-dockerhub-username/auth:latest .
    docker push your-dockerhub-username/auth:latest
    ```
    *Note: You must update the image names in the Kubernetes manifest files (`manifests/*.yaml`) to point to your Docker Hub repository.*

3.  **Deploy to Kubernetes:**
    Apply all the Kubernetes configuration files located in the `manifests` directory.
    ```bash
    kubectl apply -f manifests/
    ```

4.  **Enable Ingress:**
    The API Gateway is exposed via a Kubernetes Ingress. Enable it in Minikube:
    ```bash
    minikube addons enable ingress
    ```

5.  **Create a Tunnel:**
    Open a separate terminal window and run the following command to tunnel traffic from your local machine to the Minikube cluster:
    ```bash
    minikube tunnel
    ```
    *Keep this terminal running.*

## API Workflow

1.  **Login:** Send a `POST` request to `http://mp3converter.com/login` with basic authentication (email/password) to receive a JWT.
2.  **Upload:** Send a `POST` request to `http://mp3converter.com/upload` with a multipart/form-data video file and an `Authorization: Bearer <YOUR_JWT>` header.
3.  **Notification:** The system will process the video asynchronously. Once complete, an email notification will be sent to the user's registered email address containing the MP3's unique file ID.
4.  **Download:** Send a `GET` request to `http://mp3converter.com/download?fid=<FILE_ID_FROM_EMAIL>` with the JWT header to download the converted MP3 file.
