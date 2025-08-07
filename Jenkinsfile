pipeline {
    agent any

    tools {
        jdk 'JDK_HOME'
        maven 'MAVEN_HOME'
        nodejs 'NODE_HOME' // Include nodejs in tools directly
    }

    environment {
        BACKEND_DIR = 'crud_backend/crud_backend-main'
        FRONTEND_DIR = 'crud_frontend/crud_frontend-main'

        TOMCAT_URL = 'http://13.62.51.44:9090/manager/text'
        TOMCAT_USER = 'admin'
        TOMCAT_PASS = 'admin'

        BACKEND_WAR = 'springapp1.war'
        FRONTEND_WAR = 'frontapp1.war'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/2300031308/practical3_fullstack.git', branch: 'master'
            }
        }

        stage('Build Frontend (Vite)') {
            steps {
                dir("${env.FRONTEND_DIR}") {
                    sh 'node -v'        // Check Node version
                    sh 'npm -v'         // Check npm version
                    sh 'npm install'    // Install dependencies
                    sh 'npm run build'  // Build Vite app
                }
            }
        }

        stage('Package Frontend as WAR') {
            steps {
                dir("${env.FRONTEND_DIR}") {
                    sh """
                        mkdir -p frontapp1_war/WEB-INF
                        cp -r dist/* frontapp1_war/
                        jar -cvf ../../${FRONTEND_WAR} -C frontapp1_war .
                    """
                }
            }
        }

        stage('Build Backend (Spring Boot WAR)') {
            steps {
                dir("${env.BACKEND_DIR}") {
                    sh 'mvn clean package'
                    sh "cp target/*.war ../../${BACKEND_WAR}"
                }
            }
        }

        stage('Deploy Backend to Tomcat (/springapp1)') {
            steps {
                sh """
                    curl -u ${TOMCAT_USER}:${TOMCAT_PASS} \\
                      --upload-file ${BACKEND_WAR} \\
                      "${TOMCAT_URL}/deploy?path=/springapp1&update=true"
                """
            }
        }

        stage('Deploy Frontend to Tomcat (/frontapp1)') {
            steps {
                sh """
                    curl -u ${TOMCAT_USER}:${TOMCAT_PASS} \\
                      --upload-file ${FRONTEND_WAR} \\
                      "${TOMCAT_URL}/deploy?path=/frontapp1&update=true"
                """
            }
        }
    }

    post {
        success {
            echo "✅ Backend deployed: http://13.62.51.44:9090/springapp1"
            echo "✅ Frontend deployed: http://13.62.51.44:9090/frontapp1"
        }
        failure {
            echo "❌ Build or deployment failed"
        }
    }
}
