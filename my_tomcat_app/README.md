### **Step-by-Step Process to Deploy a Docker-based Application Using Jenkins Pipeline**

#### **1. Create the Folder and Dockerfile in Your GitHub Repository**
1. In your GitHub repository root, create a folder named **`my_tomcat_app`**.
2. Inside this folder, create the following files:
   - **Dockerfile** (example provided below)
   - **app.war** (your application file, if applicable)
   - **Jenkinsfile** (for the pipeline)

---

### **Sample Dockerfile** (`my_tomcat_app/Dockerfile`)

```dockerfile
FROM tomcat:9.0-jdk11

# Copy the application WAR file to the Tomcat webapps directory
COPY app.war /usr/local/tomcat/webapps/

# Expose the port Tomcat runs on
EXPOSE 8080

# Start Tomcat
CMD ["catalina.sh", "run"]
```

---

### **2. Create the Jenkinsfile**
Place the **Jenkinsfile** in the **`my_tomcat_app`** folder of your repository.

---

### **Jenkinsfile** (`my_tomcat_app/Jenkinsfile`)

```groovy
pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/discover-devops/JenkinsDemo.git'  // Your repo URL
        DOCKER_IMAGE = 'discoverdevops/my-tomcat-app:latest'             // Docker image name
        APP_FOLDER = 'my_tomcat_app'                                     // Folder containing the Dockerfile and other files
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                echo 'Checking out source code...'
                git branch: 'main', url: "${REPO_URL}"
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    // Specify the folder where the Dockerfile is located
                    docker.build("${DOCKER_IMAGE}", "./${APP_FOLDER}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing Docker image to DockerHub...'
                script {
                    docker.withRegistry('', 'dockerhub-credentials') {
                        docker.image(DOCKER_IMAGE).push()
                    }
                }
            }
        }

        stage('Deploy Application') {
            steps {
                echo 'Deploying application...'
                script {
                    sh """
                    docker stop my-tomcat-app || true
                    docker rm my-tomcat-app || true
                    docker run -d --name my-tomcat-app -p 8080:8080 ${DOCKER_IMAGE}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }

        failure {
            echo 'Pipeline execution failed!'
        }

        always {
            cleanWs()
        }
    }
}
```

---

### **3. Explanation of the Jenkinsfile**

1. **Pipeline Structure**:
   - The pipeline contains multiple stages: Checkout, Build, Push, and Deploy.
   - It is designed to automatically build and deploy your Docker-based Tomcat application.

2. **Folder Path for Docker Build**:
   - The pipeline uses the `docker.build()` method to specify the path where the Dockerfile is located:
     ```groovy
     docker.build("${DOCKER_IMAGE}", "./${APP_FOLDER}")
     ```
   - The **`APP_FOLDER`** variable points to the **`my_tomcat_app`** folder, which contains the Dockerfile and application files.

3. **Credentials**:
   - The pipeline references `dockerhub-credentials` for pushing the Docker image to DockerHub. Make sure this credential exists in Jenkins.

4. **Application Deployment**:
   - The pipeline deploys the Docker container by running the following command:
     ```sh
     docker run -d --name my-tomcat-app -p 8080:8080 ${DOCKER_IMAGE}
     ```

5. **Post Actions**:
   - The pipeline includes post actions to handle both success and failure cases.
   - It also cleans up the Jenkins workspace after execution using the `cleanWs()` step.

---

### **4. Jenkins Pipeline Setup**

1. Open Jenkins and create a new pipeline job.
2. In the job configuration:
   - **Select Pipeline type**.
   - **Specify the Git repository URL**: `https://github.com/discover-devops/JenkinsDemo.git`.
   - **Set the Script Path** to `my_tomcat_app/Jenkinsfile`.

3. Save and run the pipeline.

---

### **5. Verify the Deployment**

1. Once the pipeline executes successfully, verify that the application is running.
2. Open a browser and navigate to:
   ```
   http://<docker-host-ip>:8080/app/
   ```
   Replace `<docker-host-ip>` with the IP address or hostname of the Docker host.

---

### **6. Troubleshooting Tips**

- **File Path Issues**:  
  Ensure the `Dockerfile`, `app.war`, and `Jenkinsfile` are located in the `my_tomcat_app` folder in your GitHub repository.

- **Docker Access Issues**:  
  Make sure Jenkins has permission to access Docker on the worker node.

- **Docker Credentials**:  
  Ensure the credentials (`dockerhub-credentials`) for DockerHub are correctly configured in Jenkins.

---

### **Summary**

1. Create a folder named `my_tomcat_app` in your GitHub repo.
2. Add the Dockerfile, Jenkinsfile, and application files to this folder.
3. Define the pipeline stages in the Jenkinsfile to automate the build and deployment process.
4. Configure and run the Jenkins pipeline job to deploy your application.
