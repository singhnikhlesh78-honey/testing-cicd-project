pipeline {

    agent any

    environment {
        IMAGE_NAME = "nikhleshtest/testing-cicd-project"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv jenkins-venv
                    . jenkins-venv/bin/activate

                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    . jenkins-venv/bin/activate
                    pytest -v
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    rm -rf build
                    mkdir -p build

                    cp -r app build/
                    cp -r tests build/
                    cp requirements.txt build/
                    cp pytest.ini build/

                    tar -czf testing-cicd-project.tar.gz -C build .
                '''
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'testing-cicd-project.tar.gz',
                                   fingerprint: true
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} \
                        -t ${IMAGE_NAME}:latest \
                        .
                '''
            }
        }

        stage('Docker Test') {
            steps {
                sh '''
                    docker rm -f testing-cicd-project-test || true

                    docker run -d \
                        --name testing-cicd-project-test \
                        -p 5001:5000 \
                        ${IMAGE_NAME}:${IMAGE_TAG}

                    sleep 5

                    curl --fail http://127.0.0.1:5001/health

                    docker rm -f testing-cicd-project-test
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin

                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml

                    kubectl set image deployment/testing-cicd-project \
                        testing-cicd-project=${IMAGE_NAME}:${IMAGE_TAG}

                    kubectl rollout status deployment/testing-cicd-project \
                        --timeout=120s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get deployment testing-cicd-project
                    kubectl get pods -l app=testing-cicd-project -o wide
                    kubectl get svc testing-cicd-project
                '''
            }
        }
    }

    post {

        always {
            sh '''
                docker rm -f testing-cicd-project-test 2>/dev/null || true
            '''
        }

        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed!'
        }
    }
}
