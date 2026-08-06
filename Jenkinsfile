pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        timeout(time: 45, unit: 'MINUTES')
    }

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'

        DOCKER_IMAGE = "abhi888a/ecommerce-app:latest"

        K8S_CLUSTER = "kastro-eks"
        K8S_NAMESPACE = "webapps"
        DEPLOYMENT_NAME = "ecommerce-deployment"

        K8S_SERVER = "https://43F4353F25F57F9DC7DEBFB48B039EC7.yl4.ap-south-1.eks.amazonaws.com"
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'master',
                url: 'https://github.com/ab-inand/Ecommerce-App-.git'
            }
        }

        stage('Verify Files') {
            steps {
                sh '''
                pwd
                ls -la
                '''
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {

                    sh """
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=ECommerce-App \
                    -Dsonar.projectName=ECommerce-App \
                    -Dsonar.sources=src \
                    -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time:5, unit:'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Publish to Nexus') {
            steps {
                withMaven(
                    globalMavenSettingsConfig: 'maven-setting',
                    jdk: 'jdk17',
                    maven: 'maven3'
                ) {
                    sh 'mvn deploy -DskipTests'
                }
            }
        }

        stage('Trivy File System Scan') {
            steps {
                sh '''
                trivy fs \
                --severity HIGH,CRITICAL \
                --format table \
                --output trivy-fs-report.txt \
                .
                '''

                archiveArtifacts artifacts:'trivy-fs-report.txt', fingerprint:true
            }
        }

        stage('Docker Build') {
            steps {
                script {

                    withDockerRegistry(credentialsId:'dockerhub') {

                        sh """
                        docker build -t ${DOCKER_IMAGE} .
                        """
                    }
                }
            }
        }

        stage('Trivy Docker Image Scan') {
            steps {

                sh """
                trivy image \
                --severity HIGH,CRITICAL \
                --format table \
                --output trivy-image-report.txt \
                ${DOCKER_IMAGE}
                """

                archiveArtifacts artifacts:'trivy-image-report.txt', fingerprint:true
            }
        }

        stage('Push Docker Image') {

            steps {

                script {

                    withDockerRegistry(credentialsId:'dockerhub') {

                        sh """
                        docker push ${DOCKER_IMAGE}
                        """
                    }
                }
            }
        }

        stage('Deploy To Kubernetes') {
    steps {
        sh '''
        kubectl apply -f deployment-service.yaml

        kubectl rollout status deployment/ecommerce-deployment \
        -n webapps \
        --timeout=180s
        '''
    }
}

        stage('Verify Deployment') {
    steps {
        sh '''
        echo "Pods"
        kubectl get pods -n webapps

        echo "Services"
        kubectl get svc -n webapps

        echo "Deployment"
        kubectl get deployment -n webapps
        '''
    }
}

        stage('Rolling Restart') {
    steps {
        sh '''
        kubectl rollout restart deployment ecommerce-deployment -n webapps

        kubectl rollout status deployment ecommerce-deployment -n webapps
        '''
    }
}
    }
}
