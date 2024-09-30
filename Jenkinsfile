pipeline {
    agent any

    environment {
        GITHUB_REPO = "https://github.com/AhmedFadel0393/chakra-ui.git"
        RECIPIENTS = "developer@example.com"
        GIT_USER_NAME = "AhmedFadel0393"
        GIT_USER_EMAIL = "ahmed.fadel0393@gmail.com"
        NODE_VERSION = "18.6.0" // Ensure the pipeline uses Node.js v18.6.0
    }

    stages {
        stage('Ensure Node.js v18.6.0 is Installed') {
            steps {
                script {
                    // Check if Node.js v18.6.0 is installed; if not, install it
                    def installedNodeVersion = sh(script: 'node -v', returnStdout: true).trim()
                    if (installedNodeVersion != "v${NODE_VERSION}") {
                        // Install Node.js v18.6.0
                        sh '''
                            curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
                            sudo apt-get install -y nodejs
                            node -v
                            npm -v
                        '''
                    } else {
                        echo "Node.js v18.6.0 is already installed: ${installedNodeVersion}"
                    }
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
                    script {
                        if (fileExists('.git')) {
                            sh 'git pull https://$GITHUB_TOKEN@github.com/AhmedFadel0393/chakra-ui.git'
                        } else {
                            sh 'git clone https://$GITHUB_TOKEN@github.com/AhmedFadel0393/chakra-ui.git .'
                        }
                    }
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install @chakra-ui/react @emotion/react @emotion/styled framer-motion'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Project') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Version and Tag Release') {
            steps {
                script {
                    def currentVersion = sh(script: "npm version patch --no-git-tag-version", returnStdout: true).trim()
                    sh "git tag -a v${currentVersion} -m 'Release version ${currentVersion}'"
                    withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                        sh 'git push https://$GITHUB_TOKEN@github.com/AhmedFadel0393/chakra-ui.git v${currentVersion}'
                    }
                }
            }
        }

        stage('Push to GitHub') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "${GIT_USER_EMAIL}"
                        git config user.name "${GIT_USER_NAME}"
                        git add .
                        git commit -m "Automated Build & Test - New Version"
                        git push https://$GITHUB_TOKEN@github.com/AhmedFadel0393/chakra-ui.git
                    '''
                }
            }
        }
    }

    post {
        success {
            emailext (
                subject: "Build Successful: UI Library - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>Good news!</p>
                         <p>The build for <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> was successful.</p>
                         <p>Check the latest version at: <a href="${GITHUB_REPO}">GitHub Repository</a></p>
                         <p>Jenkins Build URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>""",
                to: "${RECIPIENTS}",
                mimeType: 'text/html'
            )
        }
        failure {
            emailext (
                subject: "Build Failed: UI Library - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>Attention Required!</p>
                         <p>The build for <b>${env.JOB_NAME} #${env.BUILD_NUMBER}</b> has failed.</p>
                         <p>Please check the logs to identify and fix the issue.</p>
                         <p>Jenkins Build URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>""",
                to: "${RECIPIENTS}",
                mimeType: 'text/html'
            )
        }
    }
}
