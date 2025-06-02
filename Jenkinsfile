// Jenkinsfile
// This defines a comprehensive CI/CD pipeline for a Java Maven project.
// It includes stages for source checkout, build, test, and deployment,
// along with post-build actions, environment variables, and conditional steps.

pipeline {
    // Define the agent where the pipeline will run.
    // 'any' means Jenkins will use any available agent.
    // For more control and reproducible environments, consider a Docker agent:
    // agent {
    //     docker {
    //         image 'maven:3.8.1-jdk-11' // Use a Docker image with Maven and JDK
    //         args '-v /var/run/docker.sock:/var/run/docker.sock' // Required if you build Docker images within the pipeline
    //     }
    // }
    agent any

    // Define global environment variables for the pipeline.
    // These variables can be accessed throughout the pipeline using ${VAR_NAME}.
    environment {
        APP_NAME = 'my-java-app' // Name of your application
        // Constructs the artifact name based on APP_NAME and Jenkins BUILD_NUMBER
        ARTIFACT_NAME = "${APP_NAME}-${BUILD_NUMBER}.jar"
        // Set a flag for production deployment (can be overridden by Jenkins parameters)
        PRODUCTION_ENABLED = 'false'
        // Replace with your actual deployment server details
        DEPLOY_SERVER_USER = 'jenkinsuser'
        DEPLOY_SERVER_HOST = 'your-server-ip-or-hostname'
        DEPLOY_SERVER_PATH = '/opt/your-app'
    }

    // Define tools required for the pipeline.
    // 'M3' should match the name of your Maven installation configured in Jenkins
    // (Manage Jenkins -> Global Tool Configuration -> Maven installations).
   tools {
    maven 'Maven 3.9.6'  // Use the exact Maven installation name you configured in Jenkins
}
    }

    // Define the stages of your CI/CD pipeline.
    stages {
        // Stage 1: Checkout Source Code
        // Jenkins automatically clones the repository configured in the job's SCM settings.
        // This stage is mainly for logging and confirming the checkout process.
        stage('Checkout Source') {
            steps {
                echo 'Cloning repository from Git...'
                // If you needed to explicitly clone a different repo or use specific git options:
                // git url: 'https://github.com/Gaddam1555/project.git', branch: 'main'
            }
        }

        // Stage 2: Build the Application
        // This stage compiles the code and packages it into an executable artifact.
        stage('Build') {
            steps {
                echo "Building the application: ${APP_NAME}..."
                // Execute Maven clean install to compile, test, and package the application.
                // -DfinalName=${ARTIFACT_NAME} ensures the output JAR has the desired name.
                sh "mvn clean install -DfinalName=${ARTIFACT_NAME}"
                // Archive the generated JAR artifact.
                // This makes the artifact available for download from the Jenkins build page.
                // 'fingerprint: true' helps track where the artifact is used.
                archiveArtifacts artifacts: "target/${ARTIFACT_NAME}", fingerprint: true
            }
        }

        // Stage 3: Run Tests
        // This stage executes automated tests (unit, integration, etc.).
        stage('Test') {
            steps {
                echo 'Running tests...'
                // Execute Maven tests.
                sh 'mvn test'
                // You might also publish test results here, e.g., for JUnit:
                // junit '**/target/surefire-reports/*.xml'
            }
        }

        // Stage 4: Manual Approval for Production Deployment
        // This stage pauses the pipeline and waits for manual approval.
        // This is crucial for controlled deployments to critical environments.
        stage('Approve for Production') {
            // Only run this stage if PRODUCTION_ENABLED is true (e.g., set via a build parameter)
            when {
                expression { env.PRODUCTION_ENABLED == 'true' }
            }
            steps {
                // The 'input' step pauses the pipeline and presents a message to the user.
                input message: 'Proceed to deploy to Production environment?'
            }
        }

        // Stage 5: Deploy to Production
        // This stage deploys the application to the production environment.
        stage('Deploy to Production') {
            // Only run this stage if PRODUCTION_ENABLED is true AND the current branch is 'main'.
            // This ensures only approved and main branch changes go to production.
            when {
                branch 'main'
                expression { env.PRODUCTION_ENABLED == 'true' }
            }
            steps {
                echo "Deploying ${ARTIFACT_NAME} to production server: ${DEPLOY_SERVER_HOST}..."
                // Use 'withCredentials' to securely access SSH username and password.
                // 'my-ssh-credentials' should be the ID of your 'Username with password' credential in Jenkins.
                withCredentials([usernamePassword(credentialsId: 'my-ssh-credentials',
                                                  usernameVariable: 'SSH_USER',
                                                  passwordVariable: 'SSH_PASS')]) {
                    // Copy the artifact to the remote server using scp.
                    // sshpass is used here for simplicity, but for production, consider SSH Agent or other secure methods.
                    // -o StrictHostKeyChecking=no is for initial connection, remove in production after host key is known.
                    sh "sshpass -p ${SSH_PASS} scp -o StrictHostKeyChecking=no target/${ARTIFACT_NAME} ${SSH_USER}@${DEPLOY_SERVER_HOST}:${DEPLOY_SERVER_PATH}/"
                    // Optionally, restart the application on the remote server
                    // sh "sshpass -p ${SSH_PASS} ssh -o StrictHostKeyChecking=no ${SSH_USER}@${DEPLOY_SERVER_HOST} 'sudo systemctl restart my-java-app'"
                }
                echo 'Deployment to production completed.'
            }
        }
    }

    // Post-build actions: These steps run after all stages are completed.
    post {
        // 'always' block: Runs regardless of the pipeline's success or failure.
        always {
            echo 'Pipeline finished.'
            // Clean up the workspace to free up disk space.
            cleanWs()
        }
        // 'success' block: Runs only if the pipeline completes successfully.
        success {
            echo 'Pipeline succeeded! Sending success notification...'
            // Example for sending an email notification:
            // mail to: 'your_email@example.com',
            //      subject: "Jenkins Build ${env.JOB_NAME} - #${env.BUILD_NUMBER} Succeeded",
            //      body: "Jenkins pipeline for ${env.APP_NAME} build #${env.BUILD_NUMBER} succeeded.\nCheck build at: ${env.BUILD_URL}"
        }
        // 'failure' block: Runs only if the pipeline fails at any stage.
        failure {
            echo 'Pipeline failed! Sending failure notification...'
            // Example for sending an email notification on failure:
            // mail to: 'your_email@example.com',
            //      subject: "Jenkins Build ${env.JOB_NAME} - #${env.BUILD_NUMBER} Failed",
            //      body: "Jenkins pipeline for ${env.APP_NAME} build #${env.BUILD_NUMBER} FAILED.\nReview logs at: ${env.BUILD_URL}"
        }
        // Other post-conditions you can use:
        // changed { echo 'Pipeline status changed from previous build.' }
        // unstable { echo 'Pipeline finished with unstable status (e.g., tests failed).' }
        // aborted { echo 'Pipeline was aborted.' }
        // fixed { echo 'Pipeline was previously failing and is now successful.' }
    }
}
