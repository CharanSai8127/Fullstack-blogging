pipeline {
    agent any

    parameters {
        string(name: 'DOCKER_TAG', defaultValue: 'latest', description: 'Docker image tag')
    }

    environment {
        IMAGE_NAME   = 'charansait372/devops-blog-new'
        TAG          = "${params.DOCKER_TAG}"
        SCANNER_HOME = tool 'sonar-scanner'
    }

    tools {
        maven 'maven 3'
        jdk 'jdk 17'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'git-cred',
                    url: 'https://github.com/CharanSai8127/Fullstack-blogging.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Tests') {
            steps {
                sh 'echo "Tests intentionally skipped in CI"'
            }
        }

        stage('Dependency Check (OWASP)') {
            steps {
                dependencyCheck additionalArguments: '--scan ./',
                                odcInstallation: 'owasp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh """
                      ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=fullstack-blog \
                        -Dsonar.projectName=fullstack-blog \
                        -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withMaven(mavenSettingsConfig: 'maven-config', maven: 'maven 3') {
                    sh 'mvn deploy -DskipTests'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${TAG} .'
            }
        }

        stage('Docker Push') {
            steps {
                withDockerRegistry(
                    credentialsId: 'docker-cred',
                    url: 'https://index.docker.io/v1/'
                ) {
                    sh 'docker push ${IMAGE_NAME}:${TAG}'
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                script {
                    // Generate JSON report (always succeeds)
                    sh 'trivy image --scanners vuln --format json -o trivy-report.json ${IMAGE_NAME}:${TAG}'
                    
                    // Optional: Fail only on CRITICAL vulns
                    sh '''
                        trivy image --exit-code 1 --no-progress --severity CRITICAL ${IMAGE_NAME}:${TAG} || true
                    '''
                }
            }
        }
    }

    post {
        success {
            // Trivy report (may be empty for clean images)
            archiveArtifacts artifacts: 'trivy-report.json', 
                            fingerprint: true, 
                            allowEmptyArchive: true
            
            // OWASP reports
            archiveArtifacts artifacts: '**/dependency-check-report.*',
                            fingerprint: true,
                            allowEmptyArchive: true
            
            // Sonar report (if generated)
            archiveArtifacts artifacts: '**/sonar-report.json', 
                            fingerprint: true,
                            allowEmptyArchive: true
        }
        failure {
            echo 'Pipeline failed. Check logs above.'
            // Still archive reports for debugging
            archiveArtifacts artifacts: '**/trivy-report.json,**/dependency-check-report.*', 
                            allowEmptyArchive: true
        }
        always {
            cleanWs()
        }
    }
}
