pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the code from the Git repository
                    checkout scm
                }
            }
        }

        stage('Build') {
            steps {
                // Build your application (e.g., npm install and npm build)
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Deploy') {
            steps {
                // Deploy your application to a web server or hosting service
                // (e.g., copy files to a remote server)
            }
        }
    }
}
