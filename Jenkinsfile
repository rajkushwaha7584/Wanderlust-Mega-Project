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
                    sh """
                    $SONAR_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=wanderlust \
                    -Dsonar.projectKey=wanderlust
                    """
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
    }
}

        stage('Trivy File System Scan') {
            steps {
                sh """
                trivy fs --format table -o trivy-fs-report.html .
                """

                archiveArtifacts artifacts: 'trivy-fs-report.html', fingerprint: true
            }
        }

        stage('Deploy Using Docker') {
            steps {
                sh """
                docker compose down || true
                docker compose up -d --build
                docker ps
                """
            }
        }

        stage('OWASP ZAP Baseline Scan') {
    steps {
        sh """
        docker run --rm \
        --add-host=host.docker.internal:host-gateway \
        -v "\$WORKSPACE:/zap/wrk/:rw" \
        ghcr.io/zaproxy/zaproxy:stable \
        zap-baseline.py \
        -t http://host.docker.internal:5173 \
        -r zap-report.html \
        -J zap-report.json \
        -I
        """

        publishHTML(target: [
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: '.',
            reportFiles: 'zap-report.html',
            reportName: 'OWASP ZAP Report'
        ])

        archiveArtifacts artifacts: 'zap-report.html,zap-report.json', fingerprint: true
    }
}
    }
}
