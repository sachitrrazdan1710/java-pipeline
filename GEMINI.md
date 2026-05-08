# Project Overview: Java CI/CD Pipeline (regapp)

This is a multi-module Maven project designed as a learning resource for DevOps practices, specifically demonstrating a full CI/CD pipeline. It consists of a Java backend and a web interface, containerized with Docker and deployed to Kubernetes.

## Technologies
- **Languages:** Java 1.7, JSP
- **Build Tool:** Maven
- **Containerization:** Docker (Tomcat base image)
- **Orchestration:** Kubernetes
- **CI/CD:** Jenkins
- **Security:** Trivy (Filesystem and Image scanning)

## Architecture
- **`server/`**: A JAR module containing the core logic and unit tests.
- **`webapp/`**: A WAR module providing the web interface (JSPs) and servlet configurations.
- **`kubernetes/`**: Contains Kubernetes manifests for Deployment and Service.
- **`gitops/`**: Contains GitOps-related Jenkins configurations.

## Building and Running

### Prerequisites
- Java 7 or higher
- Maven 3.0.3+
- Docker
- Kubernetes (minikube or similar)

### Key Commands

- **Build all modules:**
  ```bash
  mvn clean package
  ```

- **Run unit tests:**
  ```bash
  mvn test
  ```

- **Run the web application locally (via Jetty):**
  ```bash
  mvn jetty:run -pl webapp
  ```

- **Build Docker Image:**
  ```bash
  mvn clean package
  sh 'cp webapp/target/webapp.war .'
  docker build -t your-registry/regapp:tag .
  ```

- **Deploy to Kubernetes:**
  ```bash
  kubectl apply -f kubernetes/regapp-deploy.yml
  kubectl apply -f kubernetes/regapp-service.yml
  ```

## CI/CD Pipeline

The project includes a robust `Jenkinsfile` that orchestrates the following stages:

1.  **Validate Parameters:** Ensures necessary parameters like `IMAGE_TAG` are provided.
2.  **Workspace Cleanup:** Starts with a clean workspace.
3.  **Git Checkout:** Fetches the latest source code.
4.  **Trivy Filesystem Scan:** Performs a security scan on the source code.
5.  **Maven Build:** Compiles code and packages the WAR file.
6.  **Docker Build:** Builds the Docker image and copies the WAR file into it.
7.  **Trivy Image Scan:** Scans the newly built Docker image for vulnerabilities.
8.  **Docker Push:** Pushes the image to a container registry.
9.  **Post-Build:** Triggers a downstream GitOps CD pipeline on success.

## Development Conventions

- **Module Structure:** Logic should reside in the `server` module, while UI-related code goes into `webapp`.
- **Testing:** Unit tests are located in `src/test/java` within their respective modules. JUnit and Mockito are the primary testing frameworks.
- **Static Analysis:** The root `pom.xml` is configured with several reporting plugins (Checkstyle, PMD, FindBugs, JXR) which can be generated using `mvn site`.
- **Dockerization:** The `Dockerfile` uses the `tomcat:latest` image. It ensures that the built `.war` file is placed in `/usr/local/tomcat/webapps` for deployment.
