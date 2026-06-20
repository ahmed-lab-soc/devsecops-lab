pipeline {
    agent {
        kubernetes {
            label 'kaniko'
        }
    }
    stages {
        stage('Build') {
            steps {
                echo 'Building application...'
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests...'
            }
        }
        stage('Build & Push Image') {
            steps {
                container('kaniko') {
                    sh '''
                    export GOOGLE_APPLICATION_CREDENTIALS=/secret/kaniko-key.json
                    /kaniko/executor \
                      --context=`pwd` \
                      --dockerfile=Dockerfile \
                      --destination=europe-west1-docker.pkg.dev/lab-soc-dev/devsecops-lab/app:latest
                    '''
                }
            }
        }
    }
}
