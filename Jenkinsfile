pipeline {
    agent any

    environment {
        SONAR_HOME = tool 'sonar'
    }

    stages {
        stage('Git Clone Code From GitHub') {
            steps {
                git url: 'https://github.com/rajkushwaha7584/Wanderlust-Mega-Project.git', branch: 'main'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                    $SONAR_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=wanderlust \
                    -Dsonar.projectKey=wanderlust
                    '''
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck(
                    odcInstallation: 'DC',
                    additionalArguments: '''
                        --scan ./
                        --format XML
                        --format HTML
                        --out .
                        --data /var/lib/jenkins/dependency-check-data
                        --noupdate
                    '''
                )

                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'

                archiveArtifacts artifacts: 'dependency-check-report.html,dependency-check-report.xml', allowEmptyArchive: true, fingerprint: true
            }
        }

        stage('Trivy File System Scan') {
            steps {
                sh '''
                trivy fs \
                --scanners vuln \
                --skip-dirs node_modules \
                --format table \
                -o trivy-fs-report.txt .
                '''

                archiveArtifacts artifacts: 'trivy-fs-report.txt', allowEmptyArchive: true, fingerprint: true
            }
        }

        stage('Deploy Using Docker') {
            steps {
                sh '''
                docker rm -f frontend backend redis-service mongo-service || true

                docker compose down --remove-orphans || true
                docker compose up -d --build --remove-orphans

                docker ps
                '''
            }
        }

        stage('OWASP ZAP Baseline Scan') {
            steps {
                sh '''
                mkdir -p zap-reports

                docker run --rm \
                --add-host=host.docker.internal:host-gateway \
                -v "$WORKSPACE/zap-reports:/zap/wrk/:rw" \
                ghcr.io/zaproxy/zaproxy:stable \
                zap-baseline.py \
                -t http://host.docker.internal:5173 \
                -r zap-report.html \
                -w zap-report.md \
                -J zap-report.json \
                -x zap-report.xml \
                -I

                echo "===== OWASP ZAP Report Summary ====="
                sed -n '1,120p' zap-reports/zap-report.md || true
                '''

                archiveArtifacts artifacts: 'zap-reports/*', allowEmptyArchive: true, fingerprint: true
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed. Check Artifacts section for Trivy and ZAP reports.'
        }
    }
}
