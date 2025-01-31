### **Jenkins Declarative Pipeline Structure and Code**

In the previous section, we discussed the declarative pipeline. In this section, I will explain the **structure** of a declarative Jenkins pipeline in detail, including the key components and steps to define a Jenkins job through a **Jenkinsfile**.

---

### **What is a Jenkinsfile?**

A **Jenkinsfile** is a text file that contains the definition of a Jenkins pipeline. It is written in Groovy-based **Domain-Specific Language (DSL)** and follows a specific structure for defining CI/CD pipelines. The Jenkinsfile should be placed in your source code repository so that it can be version-controlled along with the application code.

---

### **Basic Structure of a Declarative Pipeline**

A declarative pipeline starts with the `pipeline` keyword and includes multiple sections, each serving a different purpose. Below is the structure of a typical Jenkins declarative pipeline:

```groovy
pipeline {
    agent any

    environment {
        MY_ENV_VAR = 'sample-value'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'make build'
            }
        }

        stage('Test') {
            steps {
                sh 'make test'
            }
        }

        stage('Deploy') {
            steps {
                sh 'make deploy'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed.'
        }

        always {
            cleanWs()  // Clean workspace after build
        }
    }
}
```

---

### **Explanation of Pipeline Sections**

#### **1. Pipeline Block**
   - The pipeline starts with the `pipeline {}` block.
   - This block contains all the necessary configuration for your Jenkins job.

---

#### **2. Agent Section**
   - Defines where the pipeline should run.
   - The keyword `any` means the job can run on any available Jenkins agent (or worker node).
   - You can also specify a specific agent label.

   **Example:**
   ```groovy
   agent {
       label 'docker-agent'
   }
   ```

---

#### **3. Environment Section**
   - Allows you to define environment variables that can be used throughout the pipeline.
   - These variables help manage settings such as paths, URLs, or configuration values.

   **Example:**
   ```groovy
   environment {
       DOCKER_IMAGE = 'my-docker-repo/my-app:latest'
   }
   ```

---

#### **4. Stages Section**
   - The core of the pipeline, where you define various **stages** of the CI/CD process.
   - Each stage contains **steps** that define what actions should be performed.
   - Stages are executed sequentially.

   **Example:**
   ```groovy
   stages {
       stage('Checkout Code') {
           steps {
               git branch: 'main', url: 'https://github.com/your-repo.git'
           }
       }

       stage('Build') {
           steps {
               sh 'make build'
           }
       }
   }
   ```

---

#### **5. Steps Section**
   - Defines the tasks to perform within a stage.
   - Common steps include shell commands (`sh`), script execution, and fetching code from a repository.

   **Example:**
   ```groovy
   steps {
       sh 'npm install'
       sh 'npm test'
   }
   ```

---

#### **6. Post Section**
   - This section defines actions to be performed after the pipeline finishes (successfully or with failure).
   - Common use cases include sending notifications or cleaning up the workspace.

   **Example:**
   ```groovy
   post {
       success {
           echo 'Build succeeded!'
       }

       failure {
           echo 'Build failed!'
       }

       always {
           cleanWs()
       }
   }
   ```

---

### **Detailed Sample Jenkinsfile**

Here's a more detailed example that includes all the core components of a declarative pipeline:

```groovy
pipeline {
    agent {
        label 'linux-agent'
    }

    environment {
        APP_NAME = 'my-app'
        DOCKER_IMAGE = "dockerhub/${APP_NAME}:latest"
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                echo 'Fetching source code...'
                git branch: 'main', url: 'https://github.com/your-repo/my-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    docker.build(DOCKER_IMAGE)
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                echo 'Running unit tests...'
                sh './run-tests.sh'
            }
        }

        stage('Push Docker Image') {
            steps {
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
                sh 'docker run -d -p 8080:8080 ${DOCKER_IMAGE}'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the logs for more details.'
        }

        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}
```

---

### **Pipeline Execution Flow**

1. **Checkout Stage**:  
   The pipeline fetches the latest code from the Git repository.

2. **Build Stage**:  
   The pipeline builds the application (e.g., compiling code, building Docker images).

3. **Test Stage**:  
   The pipeline runs unit tests or other automated tests to verify the build.

4. **Push Stage**:  
   If using Docker, the pipeline pushes the built Docker image to a Docker registry.

5. **Deploy Stage**:  
   The application is deployed to the target environment (e.g., a server or Kubernetes cluster).

6. **Post Section**:  
   After the pipeline completes, post-actions such as notifications, log analysis, and workspace cleanup are executed.

---

### **Benefits of Declarative Pipeline**

1. **Simplified Syntax**:  
   Easy to write, read, and maintain.

2. **Version Control Integration**:  
   Jenkinsfile can be stored in the same repository as the application code, ensuring both are versioned together.

3. **Reusability**:  
   A single Jenkinsfile can handle multiple branches and environments.

4. **Automation**:  
   Reduces manual intervention by automating the entire CI/CD workflow.

5. **Error Handling**:  
   Built-in mechanisms for handling pipeline failures and taking corrective actions.

---

### **Conclusion**

The **Jenkins declarative pipeline** provides a powerful and flexible way to define and automate complex build and deployment processes. 
By using a Jenkinsfile, you can manage your pipeline as code, enabling better collaboration, scalability, and version control. 
