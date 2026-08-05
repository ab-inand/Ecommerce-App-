pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
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
                sh 'pwd'
                sh 'ls -la'
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
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=ECommerce-App \
                    -Dsonar.projectName=ECommerce-App \
                    -Dsonar.sources=src \
                    -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
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
        --format table \
        --output trivy-fs-report.txt \
        .
        '''
        archiveArtifacts artifacts: 'trivy-fs-report.txt', fingerprint: true
    }
}
        stage('Docker Build') {
    steps {
        script {
            sh 'docker build -t abhi888a/ecommerce-app:latest .'
        }
    }
}
        stage('Trivy Docker Image Scan') {
    steps {
        sh '''
        trivy image \
        --severity HIGH,CRITICAL \
        --format table \
        -o trivy-image-report.txt \
        abhi888a/ecommerce-app:latest
        '''
        archiveArtifacts artifacts: 'trivy-image-report.txt', fingerprint: true
    }
}
       
    
    }
}
