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
        stage('SCA') {
            agent {
                kubernetes {
                    label 'sca'
                }
            }
            steps {
                container('dependency-check') {
                    sh '''
                    /usr/share/dependency-check/bin/dependency-check.sh \
                      --project "devsecops-lab" \
                      --scan . \
                      --format HTML \
                      --out dependency-check-report \
                      --disableAssembly
                    '''
                }
                archiveArtifacts artifacts: 'dependency-check-report/*.html', allowEmptyArchive: true
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
