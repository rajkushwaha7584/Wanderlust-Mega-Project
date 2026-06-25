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

               // archiveArtifacts artifacts: 'trivy-fs-report.html', fingerprint: true
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
    }
}
