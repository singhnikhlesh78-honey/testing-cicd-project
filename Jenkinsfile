pipeline {
    agent any

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
                    echo "Building application..."

                    rm -rf build
                    mkdir -p build

                    cp -r app build/
                    cp -r tests build/
                    cp requirements.txt build/
                    cp pytest.ini build/

                    tar -czf testing-cicd-project.tar.gz -C build .

                    echo "Build completed"
                    ls -lh testing-cicd-project.tar.gz
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
                    echo "Building Docker image..."

                    docker build \
                        -t testing-cicd-project:${BUILD_NUMBER} .

                    docker tag \
                        testing-cicd-project:${BUILD_NUMBER} \
                        testing-cicd-project:latest

                    docker images | grep testing-cicd-project
                '''
            }
        }

        stage('Docker Run') {
            steps {
                sh '''
                    echo "Starting Docker container..."

                    docker rm -f testing-cicd-project || true

                    docker run -d \
                        --name testing-cicd-project \
                        -p 5000:5000 \
                        testing-cicd-project:${BUILD_NUMBER}

                    sleep 5

                    docker ps
                '''
            }
        }

        stage('Docker Test') {
            steps {
                sh '''
                    echo "Testing application inside Docker..."

                    curl -f http://localhost:5000/
                    curl -f http://localhost:5000/health
                    curl -f http://localhost:5000/add/10/20

                    echo ""
                    echo "Docker application test successful"
                '''
            }
        }
    }

    post {
        always {
            sh '''
                echo "Cleaning temporary Docker resources..."
                docker ps -a
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
