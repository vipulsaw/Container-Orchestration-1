pipeline {
    agent any

    environment {
        REGISTRY = "tanujbhatia24"
        FRONTEND_REPO = "https://github.com/tanujbhatia24/frontend_kube.git"
        BACKEND_REPO = "https://github.com/tanujbhatia24/backend_kube.git"
        FRONTEND_IMAGE = "${REGISTRY}/frontend"
        BACKEND_IMAGE = "${REGISTRY}/backend"
    }

    stages {
        stage('Clone Repositories') {
            steps {
                script {
                    // Clone Frontend
                    dir('frontend') {
                        git branch: 'main', url: "${env.FRONTEND_REPO}"
                        sh 'git rev-parse HEAD > ../.frontend_commit'
                    }

                    // Clone Backend
                    dir('backend') {
                        git branch: 'main', url: "${env.BACKEND_REPO}"
                        sh 'git rev-parse HEAD > ../.backend_commit'
                    }

                    // Check for changes
                    script {
                        def changed = false
                        if (fileExists('.frontend_commit.old')) {
                            def oldCommit = readFile('.frontend_commit.old').trim()
                            def newCommit = readFile('.frontend_commit').trim()
                            if (oldCommit != newCommit) {
                                echo "Frontend changed."
                                changed = true
                            }
                        } else {
                            changed = true
                        }

                        if (fileExists('.backend_commit.old')) {
                            def oldCommit = readFile('.backend_commit.old').trim()
                            def newCommit = readFile('.backend_commit').trim()
                            if (oldCommit != newCommit) {
                                echo "Backend changed."
                                changed = true
                            }
                        } else {
                            changed = true
                        }

                        if (!changed) {
                            echo "No new revisions to build."
                            currentBuild.result = 'SUCCESS'
                            error('Build skipped due to no changes.')
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh "docker build -t ${FRONTEND_IMAGE}:latest ./frontend"
                sh "docker build -t ${BACKEND_IMAGE}:latest ./backend"
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh """
                        echo $PASSWORD | docker login -u $USERNAME --password-stdin
                        docker push ${FRONTEND_IMAGE}:latest
                        docker push ${BACKEND_IMAGE}:latest
                    """
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh """
                    helm upgrade --install mern-chart ./mern-chart \
                        --namespace mern \
                        --create-namespace \
                        --values ./mern-chart/values.yaml
                """
            }
        }
    }

    post {
        success {
            // Save current commits for future diff checks
            sh 'cp .frontend_commit .frontend_commit.old || true'
            sh 'cp .backend_commit .backend_commit.old || true'
        }
        always {
            cleanWs()
        }
    }
}
