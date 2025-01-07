
pipeline {
    agent any
    
    parameters {
        string(name: 'DOCKER_TAG', defaultValue: 'latest', description: 'Docker tag')
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_NAME = 'charansait372/devops-blog-new'
        TAG = "${params.DOCKER_TAG}" // Use parameter value for tag
	ARGOCD_SERVER = "a48acb9d5d1ea4a35af9b6538b512037-1549779022.ap-south-1.elb.amazonaws.com"
    }
    
    tools {
        maven 'maven 3'
    }

    stages {
        stage('Git-Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/CharanSai8127/Fullstack-blogging.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Tests') {
            steps {
                sh 'mvn test'
            }
        }
        
         stage('Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Sonar-analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Full-stack \
                        -Dsonar.projectKey=Stack-Full \
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage('Quality-gate-check') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        
        stage('Deploy') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings-file', jdk: '', maven: 'maven 3', mavenSettingsConfig: '', traceability: true) {
                        sh 'mvn deploy'
                    }
            }
        }
        
        
        
        stage('Docker Build&Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh """
                        docker build -t ${IMAGE_NAME}:${TAG} .
                        """
                    }
                }
            }
        }
    
        stage('Trivy-image-scan') {
            steps {
                sh 'trivy image --format table -o image.txt ${IMAGE_NAME}:${TAG}'
            }
        }
        
        stage('Cleanup Untracked Files') {
            steps {
                script {
                    sh 'git clean -fdx'  // Remove untracked files and directories
                }
            }
        }
        
        stage('Update Docker Image Tag in Repo') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-cred', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_TOKEN')]) {
                        sh '''
                            git config user.name "Charansai8127"
                            git config user.email "charansait372@gmail.com"
                            
                            # Authenticate GitHub
                            git remote set-url origin https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/CharanSai8127/Fullstack-blogging.git
                            
                            # Check if the image tag in deployment/backend.yml needs to be updated
                            UPDATED=false
                            file="deployment/backend.yml"
                            if [ -f "$file" ]; then
                                if grep -q "charansait372/devops-blog:${TAG}" "$file"; then
                                    echo "No tag update needed in $file"
                                else
                                    sed -i "s|charansait372/devops-blog-new:.*|charansait372/devops-blog-new:${TAG}|g" "$file"
                                    UPDATED=true
                                fi
                            fi
                            
                            if [ "$UPDATED" = true ]; then
                                git add deployment/backend.yml
                                git commit -m "Update Docker image tag to ${TAG} in backend.yml"
                                git push origin main
                            else
                                echo "No changes detected in the YAML file"
                            fi
                        '''
                    }
                }
            }
        }

        stage('Docker-Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker push ${IMAGE_NAME}:${TAG}'
                    }
                }
            }
        }
	
	stage('Trigger ArgoCD Deployment') {
            steps {
                script {
                    // Use the argocd-token instead of username/password
                    withCredentials([string(credentialsId: 'argocd-token', variable: 'ARGOCD_TOKEN')]) {
                        sh '''
                            # Login to ArgoCD using the token
                            argocd login ${ARGOCD_SERVER} --auth-token ${ARGOCD_TOKEN} --insecure
                            
                            # Sync the application to apply changes
                            argocd app sync fullstack
                        '''
                    }
                }
            }
        }
	
    }
}


