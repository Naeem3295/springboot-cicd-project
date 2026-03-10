pipeline {
    agent any

    // =============================================
    // ENVIRONMENT VARIABLES
    // =============================================
    environment {
        // Docker Hub Configuration
        DOCKER_HUB_USERNAME = 'naeem3295'      // ← আপনার Docker Hub username দিন
        DOCKER_HUB_CREDENTIALS = 'dockerhub-Final'     // ← Jenkins credential ID
        IMAGE_NAME = "${DOCKER_HUB_USERNAME}/springboot-cicd-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        LATEST_TAG = "latest"
        CONTAINER_NAME = "springboot-cicd-container"

        // App Configuration
        APP_PORT = "8080"

        // Notification Configuration
        SLACK_CHANNEL = '#all-jenkins-cicd'              // ← আপনার Slack channel
        EMAIL_RECIPIENT = 'abun1347@gmail.com'             // ← আপনার email
    }

    // =============================================
    // BUILD PARAMETERS (Jenkins Parameterized Build)
    // =============================================
    parameters {
        choice(
            name: 'DEPLOY_ENV',
            choices: ['dev', 'staging', 'prod'],
            description: 'Select deployment environment'
        )
        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: false,
            description: 'Skip unit tests (not recommended)'
        )
        booleanParam(
            name: 'PUSH_TO_DOCKERHUB',
            defaultValue: true,
            description: 'Push Docker image to Docker Hub'
        )
    }

    // =============================================
    // PIPELINE OPTIONS
    // =============================================
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
    }

    // =============================================
    // PIPELINE STAGES
    // =============================================
    stages {

        // ─── Stage 1: Checkout ───
        stage('Checkout') {
            steps {
                echo "========================================"
                echo "  Stage 1: Checking out source code"
                echo "========================================"
                checkout scm
                sh 'echo "Branch: $(git rev-parse --abbrev-ref HEAD)"'
                sh 'echo "Commit: $(git rev-parse --short HEAD)"'
            }
        }

        // ─── Stage 2: Build ───
        stage('Build') {
            steps {
                echo "========================================"
                echo "  Stage 2: Building Spring Boot App"
                echo "========================================"
                sh 'mvn clean compile -B'
            }
        }

        // ─── Stage 3: Test ───
        stage('Test') {
            when {
                expression { return !params.SKIP_TESTS }
            }
            steps {
                echo "========================================"
                echo "  Stage 3: Running Unit Tests"
                echo "========================================"
                sh 'mvn test -B'
            }
            post {
                always {
                    // Publish JUnit test results
                    junit 'target/surefire-reports/*.xml'
                    echo "Test results published"
                }
                success {
                    echo "✅ All tests passed!"
                }
                failure {
                    echo "❌ Tests failed! Check the test reports."
                }
            }
        }

        // ─── Stage 4: Package ───
        stage('Package') {
            steps {
                echo "========================================"
                echo "  Stage 4: Packaging Application"
                echo "========================================"
                sh 'mvn package -DskipTests -B'
                sh 'ls -lh target/*.jar'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        // ─── Stage 5: Docker Build ───
        stage('Docker Build') {
            steps {
                echo "========================================"
                echo "  Stage 5: Building Docker Image"
                echo "========================================"
                script {
                    sh """
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:${LATEST_TAG}
                        echo "Docker image built: ${IMAGE_NAME}:${IMAGE_TAG}"
                    """
                    sh 'docker images | grep springboot-cicd'
                }
            }
        }

        // ─── Stage 6: Docker Push ───
        stage('Docker Push') {
            when {
                expression { return params.PUSH_TO_DOCKERHUB }
            }
            steps {
                echo "========================================"
                echo "  Stage 6: Pushing to Docker Hub"
                echo "========================================"
                script {
                    withCredentials([usernamePassword(
                        credentialsId: "${DOCKER_HUB_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                            echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                            docker push ${IMAGE_NAME}:${IMAGE_TAG}
                            docker push ${IMAGE_NAME}:${LATEST_TAG}
                            docker logout
                            echo "✅ Successfully pushed to Docker Hub!"
                        """
                    }
                }
            }
        }

        // ─── Stage 7: Deploy ───
        stage('Deploy') {
            steps {
                echo "========================================"
                echo "  Stage 7: Deploying to ${params.DEPLOY_ENV}"
                echo "========================================"
                script {
                    sh """
                        # Stop and remove existing container
                        docker stop ${CONTAINER_NAME} || true
                        docker rm ${CONTAINER_NAME} || true

                        # Run new container
                        docker run -d \\
                            --name ${CONTAINER_NAME} \\
                            -p ${APP_PORT}:8080 \\
                            --restart unless-stopped \\
                            ${IMAGE_NAME}:${IMAGE_TAG}

                        echo "✅ Container deployed successfully!"
                        docker ps | grep ${CONTAINER_NAME}
                    """

                    // Wait for app to start
                    sleep(time: 15, unit: 'SECONDS')

                    // Health check
                    sh """
                        curl -f http://localhost:${APP_PORT}/api/health || \
                        (echo "❌ Health check failed!" && exit 1)
                        echo "✅ Health check passed!"
                    """
                }
            }
        }

        // ─── Stage 8: Clean Up ───
        stage('Clean Up') {
            steps {
                echo "========================================"
                echo "  Stage 8: Cleaning up old images"
                echo "========================================"
                sh '''
                    docker image prune -f
                    echo "Workspace cleaned"
                '''
            }
        }
    }

    // =============================================
    // POST BUILD ACTIONS
    // =============================================
    post {

        // ─── SUCCESS ───
        success {
            echo "🎉 Pipeline completed SUCCESSFULLY!"

            // Email Notification - Success
            emailext(
                subject: "✅ SUCCESS: ${JOB_NAME} - Build #${BUILD_NUMBER}",
                body: """
                    <html>
                    <body>
                        <h2 style="color: green;">✅ Build Successful!</h2>
                        <table border="1" cellpadding="5">
                            <tr><td><b>Job Name</b></td><td>${JOB_NAME}</td></tr>
                            <tr><td><b>Build Number</b></td><td>#${BUILD_NUMBER}</td></tr>
                            <tr><td><b>Environment</b></td><td>${params.DEPLOY_ENV}</td></tr>
                            <tr><td><b>Docker Image</b></td><td>${IMAGE_NAME}:${IMAGE_TAG}</td></tr>
                            <tr><td><b>Build URL</b></td><td><a href="${BUILD_URL}">${BUILD_URL}</a></td></tr>
                            <tr><td><b>Duration</b></td><td>${currentBuild.durationString}</td></tr>
                        </table>
                        <p>Spring Boot app deployed successfully on port ${APP_PORT}</p>
                    </body>
                    </html>
                """,
                to: "${EMAIL_RECIPIENT}",
                mimeType: 'text/html'
            )

            // Slack Notification - Success
            slackSend(
                channel: "${SLACK_CHANNEL}",
                color: 'good',
                message: """
                ✅ *BUILD SUCCESS*
                *Job:* ${JOB_NAME}
                *Build:* #${BUILD_NUMBER}
                *Environment:* ${params.DEPLOY_ENV}
                *Image:* ${IMAGE_NAME}:${IMAGE_TAG}
                *Duration:* ${currentBuild.durationString}
                *URL:* ${BUILD_URL}
                """
            )
        }

        // ─── FAILURE ───
        failure {
            echo "❌ Pipeline FAILED!"

            // Email Notification - Failure
            emailext(
                subject: "❌ FAILED: ${JOB_NAME} - Build #${BUILD_NUMBER}",
                body: """
                    <html>
                    <body>
                        <h2 style="color: red;">❌ Build Failed!</h2>
                        <table border="1" cellpadding="5">
                            <tr><td><b>Job Name</b></td><td>${JOB_NAME}</td></tr>
                            <tr><td><b>Build Number</b></td><td>#${BUILD_NUMBER}</td></tr>
                            <tr><td><b>Failed Stage</b></td><td>${currentBuild.result}</td></tr>
                            <tr><td><b>Build URL</b></td><td><a href="${BUILD_URL}">${BUILD_URL}</a></td></tr>
                        </table>
                        <p>Please check the console output for details.</p>
                    </body>
                    </html>
                """,
                to: "${EMAIL_RECIPIENT}",
                mimeType: 'text/html'
            )

            // Slack Notification - Failure
            slackSend(
                channel: "${SLACK_CHANNEL}",
                color: 'danger',
                message: """
                ❌ *BUILD FAILED*
                *Job:* ${JOB_NAME}
                *Build:* #${BUILD_NUMBER}
                *URL:* ${BUILD_URL}
                Please check console output!
                """
            )
        }

        // ─── ALWAYS ───
        always {
            cleanWs()
            echo "Pipeline finished. Workspace cleaned."
        }
    }
}
