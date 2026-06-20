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
        stage('Debug Paths') {
            steps {
                container('jnlp') {
                    sh 'echo "=== jnlp pwd ===" && pwd && ls -la'
                }
                container('kaniko') {
                    sh 'echo "=== kaniko pwd ===" && pwd && ls -la'
                }
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
