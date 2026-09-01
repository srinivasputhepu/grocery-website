pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 10, unit: 'MINUTES')
    }

    parameters {
        string(
            name: 'VERSION',
            defaultValue: '2.0',
            description: 'Application Version'
        )

        choice(
            name: 'ENVIRONMENT',
            choices: ['Development', 'Testing', 'Production'],
            description: 'Select an Environment'
        )
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build \
                    -t srinivasputhepu/grocery-website:${env.BUILD_NUMBER} \
                    -t srinivasputhepu/grocery-website:latest .
                """
            }
        }

        stage('Docker Push Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh """
                        echo "\$DOCKER_TOKEN" | docker login -u "\$DOCKER_USER" --password-stdin

                        docker push srinivasputhepu/grocery-website:${env.BUILD_NUMBER}
                        docker push srinivasputhepu/grocery-website:latest

                        docker logout
                    """
                }
            }
        }

        stage('EC2 Connection Test') {
            steps {
                sshagent(['aws-ec2-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                            ubuntu@13.203.196.98 \
                            "echo Successfully connected to EC2 && hostname"
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline Finished'
        }

        success {
            echo 'Pipeline Succeeded'
        }

        failure {
            echo 'Pipeline Failed'
        }
    }
}
