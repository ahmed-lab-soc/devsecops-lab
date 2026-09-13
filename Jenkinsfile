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
                    export NVD_API_KEY=$(cat /secret-nvd/NVD_API_KEY)
                    /usr/share/dependency-check/bin/dependency-check.sh \
                      --project "devsecops-lab" \
                      --scan . \
                      --format HTML \
                      --out dependency-check-report \
                      --nvdApiKey "$NVD_API_KEY" \
                      --enableExperimental \
                      --disableAssembly || true

                    if [ -f dependency-check-report/dependency-check-report.html ]; then
                      echo "Rapport genere avec succes malgre les warnings"
                      exit 0
                    else
                      echo "Echec reel - rapport non genere"
                      exit 1
                    fi
                    '''
                }
                archiveArtifacts artifacts: 'dependency-check-report/*.html', allowEmptyArchive: true
            }
        }
        stage('SAST') {
            agent {
                kubernetes {
                    label 'sast'
                }
            }
            steps {
                container('slscan') {
                    sh '''
                    scan --type python --src . --out sast-report || true
                    '''
                }
                archiveArtifacts artifacts: 'sast-report/**', allowEmptyArchive: true
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
        stage('Scan Image') {
            agent {
                kubernetes {
                    label 'trivy'
                }
            }
            steps {
                container('trivy') {
                    sh '''
                    export GOOGLE_APPLICATION_CREDENTIALS=/secret/kaniko-key.json
                    trivy image --format table --output trivy-report.txt europe-west1-docker.pkg.dev/lab-soc-dev/devsecops-lab/app:latest || true
                    cat trivy-report.txt
                    '''
                }
                archiveArtifacts artifacts: 'trivy-report.txt', allowEmptyArchive: true
            }
        }
        stage('DAST') {
            agent {
                kubernetes {
                    label 'zap'
                }
            }
            steps {
                container('zap') {
                    sh '''
                    mkdir -p /zap/wrk
                    zap-baseline.py -t http://devsecops-app-service -r /zap/wrk/zap-report.html || true
                    cp /zap/wrk/zap-report.html zap-report.html || true
                    '''
                }
                archiveArtifacts artifacts: 'zap-report.html', allowEmptyArchive: true
            }
        }
        stage('Compliance') {
            agent {
                kubernetes {
                    label 'compliance'
                }
            }
            steps {
                container('inspec') {
                    sh '''
                    inspec exec https://github.com/dev-sec/linux-baseline --reporter html:compliance-report.html || true
                    '''
                }
                archiveArtifacts artifacts: 'compliance-report.html', allowEmptyArchive: true
            }
        }
    }
}
