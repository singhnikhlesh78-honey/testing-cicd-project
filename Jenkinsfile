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
    }
}
