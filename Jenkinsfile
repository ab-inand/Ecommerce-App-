pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
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

    }
}
