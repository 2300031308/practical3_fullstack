pipeline {
    agent any

    tools {
        jdk 'JDK_HOME'
        maven 'MAVEN_HOME'
        // No nodejs here; assuming it's installed on Mac OS via Homebrew
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

        stage('Debug NPM Installation') {
            steps {
                sh '''
                    echo "Checking Node.js and npm..."
                    which node
                    node -v
                    which npm
                    npm -v
                '''
            }
        }

        stage('Build Frontend (Vite)') {
            steps {
                dir("${env.FRONTEND_DIR}") {
                    sh '''
                        echo "Installing frontend dependencies..."
                        npm install
                        echo "Building frontend..."
                        npm run build
                    '''
                }
            }
        }

        stage('Package Frontend as WAR') {
            steps {
                dir("${env.FRONTEND_DIR}") {
                    sh '''
                        echo "Packaging frontend as WAR..."
                        mkdir -p frontapp1_war/WEB-INF
                        cp -r dist/* frontapp1_war/
                        jar -cvf ../../${FRONTEND_WAR} -C frontapp1_war .
                    '''
                }
            }
        }

        stage('Build Backend (Spring Boot WAR)') {
            steps {
                dir("${env.BACKEND_DIR}") {
                    sh '''
                        echo "Building backend..."
                        mvn clean package
                        cp target/*.war ../../${BACKEND_WAR}
                    '''
                }
            }
        }

        stage('Deploy Backend to Tomcat (/springapp1)') {
            steps {
                sh '''
                    echo "Deploying backend to Tomcat..."
                    curl -u ${TOMCAT_USER}:${TOMCAT_PASS} \
                         --upload-file ${BACKEND_WAR} \
                         "${TOMCAT_URL}/deploy?path=/springapp1&update=true"
                '''
            }
        }

        stage('Deploy Frontend to Tomcat (/frontapp1)') {
            steps {
                sh '''
                    echo "Deploying frontend to Tomcat..."
                    curl -u ${TOMCAT_USER}:${TOMCAT_PASS} \
                         --upload-file ${FRONTEND_WAR} \
                         "${TOMCAT_URL}/deploy?path=/frontapp1&update=true"
                '''
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
