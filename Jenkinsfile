pipeline {
    agent any

    environment {
        NODE_VERSION = 'v18.6.0'
        NVM_DIR = '/var/lib/jenkins/.nvm' // Adjust this path if necessary
        GITHUB_CREDENTIALS = credentials('github-token')
    }

    stages {
        stage('Declarative: Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Ensure Node.js v18.6.0 is Installed') {
            steps {
                script {
                    // Install nvm and Node.js
                    sh '''
                        # Install nvm if not installed
                        if [ ! -s "$NVM_DIR/nvm.sh" ]; then
                            curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
                        fi
                        
                        # Load nvm and install the required Node.js version
                        . "$NVM_DIR/nvm.sh"
                        nvm install ${NODE_VERSION}
                        nvm use ${NODE_VERSION}
                        
                        # Verify node and npm versions
                        node -v
                        npm -v
                    '''
                }
            }
        }

        stage('Increase Git Buffer Size') {
            steps {
                sh 'git config --global http.postBuffer 1048576000'
                sh 'git config --global core.compression 1'
            }
        }

        stage('Checkout Code') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh 'git pull https://$GITHUB_TOKEN@github.com/AhmedFadel0393/chakra-ui.git'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    . "$NVM_DIR/nvm.sh"
                    nvm use ${NODE_VERSION}
                    npm install
                    npm install @chakra-ui/react @emotion/react @emotion/styled framer-motion
                    npm install @ark-ui/react
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    . "$NVM_DIR/nvm.sh"
                    nvm use ${NODE_VERSION}
                    npm test
                '''
            }
        }

        stage('Build Project') {
            steps {
                sh '''
                    . "$NVM_DIR/nvm.sh"
                    nvm use ${NODE_VERSION}
                    npm run build
                '''
            }
        }

        stage('Version and Tag Release') {
            steps {
                script {
                    // Example: Version and tag release can be customized
                    sh 'git tag -a v1.0.0 -m "Release v1.0.0"'
                    sh 'git push origin --tags'
                }
            }
        }

        stage('Push to GitHub') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh 'git push https://$GITHUB_TOKEN@github.com/AhmedFadel0393/chakra-ui.git'
                }
            }
        }
    }

    post {
        always {
            emailext(
                to: 'developer@example.com',
                subject: "Jenkins Pipeline: ${currentBuild.fullDisplayName}",
                body: "Jenkins build completed with status: ${currentBuild.currentResult}. Check Jenkins for details."
            )
        }
        success {
            echo 'Pipeline succeeded.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
