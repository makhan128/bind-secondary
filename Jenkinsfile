pipeline {
    agent { label 'podman-agent' }

    triggers {
        // Polls GitHub every 2 minutes for new commits
        pollSCM('H/2 * * * *')
    }

    environment {
        REGISTRY = '192.168.113.203:5000'
        IMAGE_NAME = 'bind-secondary'
    }

    stages {
        stage('Build Image') {
            steps {
                // Builds the container image using podman build
                sh "podman build -t ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} -t ${REGISTRY}/${IMAGE_NAME}:latest ."
            }
        }

        stage('Push to Private Registry') {
            steps {
                // Injects registry login details securely from Jenkins credentials
                withCredentials([usernamePassword(credentialsId: 'registry-creds', usernameVariable: 'REG_USER', passwordVariable: 'REG_PASS')]) {
                    sh """
                        echo "\$REG_PASS" | podman login ${REGISTRY} --username "\$REG_USER" --password-stdin --tls-verify=false
                        podman push --tls-verify=false ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}
                        podman push --tls-verify=false ${REGISTRY}/${IMAGE_NAME}:latest
                        podman logout ${REGISTRY}
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
