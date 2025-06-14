# Jenkins Complete Guide

## 1. Core Concepts

### 1.1 Jenkins Architecture
- Master-Slave Architecture
- Distributed Builds
- Plugin Ecosystem
- Security Realms
- Job Types
- Build Agents

### 1.2 Pipeline Types
1. Declarative Pipeline
   - Structured format
   - Predefined syntax
   - Easy to read and maintain

2. Scripted Pipeline
   - Groovy-based
   - More flexibility
   - Advanced programming features

3. Multibranch Pipeline
   - Automatic branch discovery
   - Different configurations per branch
   - Pull request handling

## 2. Pipeline Syntax

### 2.1 Basic Declarative Pipeline
```groovy
pipeline {
    agent any
    
    environment {
        MAVEN_HOME = tool 'Maven'
        JAVA_HOME = tool 'JDK11'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/example/repo.git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Deploy') {
            when {
                branch 'master'
            }
            steps {
                sh './deploy.sh'
            }
        }
    }
    
    post {
        success {
            emailext body: 'Build successful!',
                     subject: 'Build Status',
                     to: 'team@example.com'
        }
        failure {
            emailext body: 'Build failed!',
                     subject: 'Build Status',
                     to: 'team@example.com'
        }
    }
}
```

### 2.2 Advanced Pipeline Features
```groovy
pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: maven
                    image: maven:3.8.1-jdk-11
                    command:
                    - cat
                    tty: true
                  - name: docker
                    image: docker:19.03
                    command:
                    - cat
                    tty: true
                    volumeMounts:
                    - mountPath: /var/run/docker.sock
                      name: docker-sock
                  volumes:
                  - name: docker-sock
                    hostPath:
                      path: /var/run/docker.sock
            '''
        }
    }
    
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                container('docker') {
                    sh 'docker build -t myapp:${BUILD_NUMBER} .'
                }
            }
        }
    }
}
```

## 3. Jenkins Configuration

### 3.1 Security Setup
```groovy
// Security configuration in Groovy
jenkins.security.ApiTokenProperty.adminAndManagerOnlyMode=true
jenkins.security.csrf.DefaultCrumbIssuer.EXCLUDE_SESSION_ID=true

// LDAP configuration
jenkins.securityRealm = new hudson.security.LDAPSecurityRealm(
    server: "ldap://ldap.example.com:389",
    rootDN: "dc=example,dc=com",
    userSearchBase: "ou=users",
    userSearch: "uid={0}",
    groupSearchBase: "ou=groups"
)
```

### 3.2 Tool Configuration
```xml
<hudson.tasks.Maven_-DescriptorImpl>
  <installations>
    <hudson.tasks.Maven_-MavenInstallation>
      <name>Maven</name>
      <home>/usr/share/maven</home>
      <properties>
        <hudson.tools.InstallSourceProperty>
          <installers>
            <hudson.tasks.Maven_-MavenInstaller>
              <id>3.8.1</id>
            </hudson.tasks.Maven_-MavenInstaller>
          </installers>
        </hudson.tools.InstallSourceProperty>
      </properties>
    </hudson.tasks.Maven_-MavenInstallation>
  </installations>
</hudson.tasks.Maven_-DescriptorImpl>
```

## 4. Real-World Use Cases

### 4.1 Microservices Deployment Pipeline
```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'registry.example.com'
        SERVICE_NAME = 'user-service'
        KUBE_CONFIG = credentials('kubeconfig')
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Code Analysis') {
            parallel {
                stage('SonarQube') {
                    steps {
                        withSonarQubeEnv('SonarQube') {
                            sh 'mvn sonar:sonar'
                        }
                    }
                }
                stage('Security Scan') {
                    steps {
                        sh 'mvn dependency-check:check'
                    }
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/${SERVICE_NAME}:${BUILD_NUMBER}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'registry-credentials') {
                        docker.image("${DOCKER_REGISTRY}/${SERVICE_NAME}:${BUILD_NUMBER}").push()
                    }
                }
            }
        }
        
        stage('Deploy to Staging') {
            when { branch 'develop' }
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh """
                        kubectl apply -f k8s/staging/
                        kubectl set image deployment/${SERVICE_NAME} \
                            ${SERVICE_NAME}=${DOCKER_REGISTRY}/${SERVICE_NAME}:${BUILD_NUMBER} \
                            -n staging
                    """
                }
            }
        }
        
        stage('Integration Tests') {
            when { branch 'develop' }
            steps {
                sh 'mvn verify -Pintegration-tests'
            }
        }
        
        stage('Deploy to Production') {
            when { branch 'master' }
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input message: 'Deploy to production?'
                }
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh """
                        kubectl apply -f k8s/production/
                        kubectl set image deployment/${SERVICE_NAME} \
                            ${SERVICE_NAME}=${DOCKER_REGISTRY}/${SERVICE_NAME}:${BUILD_NUMBER} \
                            -n production
                    """
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            slackSend channel: '#deployments',
                      color: 'good',
                      message: "Deployment successful: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
        failure {
            slackSend channel: '#deployments',
                      color: 'danger',
                      message: "Deployment failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
    }
}
```

### 4.2 Frontend Application Pipeline
```groovy
pipeline {
    agent {
        docker {
            image 'node:14'
            args '-p 3000:3000'
        }
    }
    
    environment {
        CI = 'true'
        FIREBASE_TOKEN = credentials('firebase-token')
    }
    
    stages {
        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }
        
        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test -- --coverage'
            }
            post {
                always {
                    junit 'coverage/junit.xml'
                    publishHTML target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'coverage',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ]
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        
        stage('Deploy') {
            when { branch 'master' }
            steps {
                sh 'npm install -g firebase-tools'
                sh 'firebase deploy --token "$FIREBASE_TOKEN"'
            }
        }
    }
}
```

## 5. Best Practices

### 5.1 Pipeline Organization
1. Shared Libraries
```groovy
// vars/buildJavaProject.groovy
def call(Map config) {
    pipeline {
        agent any
        stages {
            stage('Build') {
                steps {
                    sh "${config.buildCommand ?: 'mvn clean package'}"
                }
            }
            stage('Test') {
                steps {
                    sh "${config.testCommand ?: 'mvn test'}"
                }
            }
        }
    }
}
```

2. Modular Stages
```groovy
// Jenkinsfile
@Library('my-shared-library') _

buildJavaProject(
    buildCommand: 'gradle build',
    testCommand: 'gradle test'
)
```

### 5.2 Security Best Practices
```groovy
pipeline {
    agent any
    
    environment {
        // Use credentials binding
        AWS_CREDENTIALS = credentials('aws-credentials')
        
        // Mask sensitive values
        SENSITIVE_VALUE = credentials('sensitive-value')
    }
    
    options {
        // Prevent concurrent builds
        disableConcurrentBuilds()
        
        // Clean workspace
        cleanWs()
    }
}
```

## 6. Monitoring and Maintenance

### 6.1 System Monitoring
```groovy
// Monitoring configuration
jenkins.model.Jenkins.instance.setNumExecutors(10)
jenkins.model.Jenkins.instance.setSlaveAgentPort(50000)

// Resource limits
jenkins.model.Jenkins.instance.setQuietPeriod(5)
jenkins.model.Jenkins.instance.setScmCheckoutRetryCount(3)
```

### 6.2 Job Monitoring
```groovy
pipeline {
    agent any
    
    options {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    
    stages {
        stage('Monitor') {
            steps {
                script {
                    def buildTime = currentBuild.durationString
                    def buildStatus = currentBuild.result
                    
                    // Send metrics to monitoring system
                    influxdb(
                        target: 'jenkins-metrics',
                        measurements: [
                            'build_time': buildTime,
                            'build_status': buildStatus
                        ]
                    )
                }
            }
        }
    }
}
```

## 7. Troubleshooting Guide

### 7.1 Common Issues
```groovy
pipeline {
    agent any
    
    stages {
        stage('Troubleshoot') {
            steps {
                script {
                    // Check system status
                    sh 'java -version'
                    sh 'mvn -version'
                    
                    // Check disk space
                    sh 'df -h'
                    
                    // Check memory usage
                    sh 'free -m'
                    
                    // Check running processes
                    sh 'ps aux | grep java'
                }
            }
        }
    }
}
```

### 7.2 Pipeline Debugging
```groovy
pipeline {
    agent any
    
    stages {
        stage('Debug') {
            steps {
                script {
                    // Print environment variables
                    sh 'printenv'
                    
                    // Print workspace info
                    echo "Workspace: ${WORKSPACE}"
                    
                    // Print build info
                    echo "Build URL: ${BUILD_URL}"
                    echo "Job Name: ${JOB_NAME}"
                    echo "Build Number: ${BUILD_NUMBER}"
                }
            }
        }
    }
}
``` 
