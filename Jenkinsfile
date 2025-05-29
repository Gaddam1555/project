# project
pipeline {
    agent any
    environment {
        APP_NAME = 'demoproject'
        ARTIFACT_NAME = "${APP_NAME}-${BUILD_NUMBER}.jar" // Example for Java
    }
    stages {
        // ...
        stage('Build') {
            steps {
                echo "Building ${APP_NAME}..."
                // sh "mvn clean package -DfinalName=${ARTIFACT_NAME}"
            }
        }
        // ...
    }
}
stage('Deploy') {
    steps {
        withCredentials([usernamePassword(credentialsId: 'my-ssh-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
            sh "sshpass -p ${PASS} scp -o StrictHostKeyChecking=no target/${ARTIFACT_NAME} ${USER}@your-server:/path/to/deploy"
        }
    }
}
stage('Build') {
    steps {
        // ... build commands ...
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
    }
}
pipeline {
    agent {
        docker {
            image 'maven:3.8.1-jdk-11' // Or node:18, python:3.9, etc.
            args '-v /var/run/docker.sock:/var/run/docker.sock' // If building Docker images
        }
    }
    // ...
}
pipeline {
    agent any
    tools {
        maven 'M3' // 'M3' should match the name configured in Global Tool Configuration
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean install' // Maven command now available
            }
        }
    }
}
stage('Deploy to Production') {
    when {
        branch 'main' // Deploy only when changes are pushed to 'main'
        environment name: 'PRODUCTION_ENABLED', value: 'true' // Or an environment variable
    }
    steps {
        echo 'Deploying to production...'
    }
}
stage('Approve for Production') {
    steps {
        input message: 'Proceed to deploy to production?', ok: 'Deploy'
    }
}
stage('Deploy to Production') {
    steps {
        echo 'Deploying to production after approval...'
    }
}
