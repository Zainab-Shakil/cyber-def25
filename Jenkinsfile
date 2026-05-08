// ╔══════════════════════════════════════════════════════════╗
// ║  Jenkinsfile  –  CYBER-DEF25 Malware Detection Pipeline  ║
// ╚══════════════════════════════════════════════════════════╝
pipeline {
    agent any
 
    // ── Global environment variables ──────────────────────
    environment {
        DOCKER_HUB_USER  = 'Zainabshakil'
        IMAGE_NAME       = 'cyber-def25'
        IMAGE_TAG        = "${IMAGE_NAME}:${BUILD_NUMBER}"
        IMAGE_LATEST     = "${IMAGE_NAME}:latest"
        DOCKER_HUB_CRED  = 'dockerhub-credentials'
    }
 
    stages {
 
        // ══════════════════════════════════════════
        // STAGE 1 – Checkout Code from GitHub
        // ══════════════════════════════════════════
        stage('Checkout') {
            steps {
                echo 'Cloning repository from GitHub...'
                git branch: 'main',
                    url: 'https://github.com/Zainab-Shakil/cyber-def25'
                echo 'Code checkout complete.'
            }
        }
 
        // ══════════════════════════════════════════
        // STAGE 2 – Build Docker Image
        // Builds the image from the Dockerfile
        // ══════════════════════════════════════════
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${DOCKER_HUB_USER}/${IMAGE_TAG}"
                script {
                    // Build with both a versioned tag and :latest tag
                    sh """
                        docker build \\
                            -t ${DOCKER_HUB_USER}/${IMAGE_TAG} \\
                            -t ${DOCKER_HUB_USER}/${IMAGE_LATEST} \\
                            .
                    """
                    echo 'Docker image built successfully.'
                    sh 'docker images | grep ${IMAGE_NAME}'
                }
            }
            post {
                failure {
                    echo 'ERROR: Docker build failed. Check Dockerfile and dependencies.'
                }
            }
        }
 
        // ══════════════════════════════════════════
        // STAGE 3 – Push Docker Image to Docker Hub
        // Uses stored Jenkins credentials for login
        // ══════════════════════════════════════════
        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing image to Docker Hub...'
                script {
                    // withDockerRegistry safely injects Docker Hub credentials
                    withDockerRegistry([
                        credentialsId: DOCKER_HUB_CRED,
                        url: 'https://index.docker.io/v1/'
                    ]) {
                        sh 'docker push ${DOCKER_HUB_USER}/${IMAGE_TAG}'
                        sh 'docker push ${DOCKER_HUB_USER}/${IMAGE_LATEST}'
                    }
                    echo 'Image pushed successfully to Docker Hub.'
                }
            }
            post {
                failure {
                    echo 'ERROR: Push failed. Check Docker Hub credentials and connectivity.'
                }
            }
        }
 
        // ══════════════════════════════════════════
        // STAGE 4 – Run Container via Docker Compose
        // Mounts ./network_logs/ and executes inference
        // ══════════════════════════════════════════
        stage('Run with Docker Compose') {
            steps {
                echo 'Running malware detection inference via Docker Compose...'
                sh """
                    # Create output directory on host if it does not exist
                    mkdir -p output
 
                    # Export Docker Hub username so Compose can resolve the image
                    export DOCKER_HUB_USER=${DOCKER_HUB_USER}
 
                    # Bring down any previous run
                    docker-compose down --remove-orphans || true
 
                    # Pull the latest image (already pushed above)
                    docker-compose pull
 
                    # Run inference (container exits after completion)
                    docker-compose up --abort-on-container-exit
 
                    echo 'Inference complete. Check ./output/alerts.csv for results.'
                """
            }
            post {
                always {
                    echo 'Cleaning up compose containers...'
                    sh 'docker-compose down || true'
                }
                failure {
                    echo 'ERROR: Docker Compose run failed. Check logs above.'
                }
            }
        }
 
    } // end stages
 
    // ── Post-pipeline notifications ───────────────────────
    post {
        success {
            echo "Pipeline SUCCESS: Image ${DOCKER_HUB_USER}/${IMAGE_TAG} built, pushed, and run."
        }
        failure {
            echo 'Pipeline FAILED. Review stage logs above for details.'
        }
        always {
            // Archive the alerts.csv output as a Jenkins build artifact
            archiveArtifacts artifacts: 'output/alerts.csv',
                             allowEmptyArchive: true
            echo 'Pipeline execution finished.'
        }
    }
}
