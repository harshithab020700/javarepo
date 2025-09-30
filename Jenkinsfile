pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/harshithab020700/javarepo.git'
        GIT_BRANCH = 'main'
        MAVEN_HOME = tool 'Maven3'       // Set up in Jenkins Global Tool Configuration
        TOMCAT_USER = 'ubuntu'
        TOMCAT_HOST = '98.81.60.180'
        TOMCAT_WEBAPPS = '/home/ubuntu/apache-tomcat-9.0.109/webapps'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Build with Maven') {
            steps {
                sh """
                    ${MAVEN_HOME}/bin/mvn clean package -DskipTests
                """
            }
        }

        stage('Check Tomcat Status') {
            steps {
                sh """
                if ssh ${TOMCAT_USER}@${TOMCAT_HOST} 'systemctl is-active --quiet tomcat'; then
                    echo 'Tomcat is running'
                else
                    echo 'Tomcat is not running'
                fi
                """
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh """
                    WAR_FILE=\$(ls target/*.war | head -n 1)
                    echo "Deploying \$WAR_FILE to Tomcat..."
                    scp -o StrictHostKeyChecking=no \$WAR_FILE ${TOMCAT_USER}@${TOMCAT_HOST}:${TOMCAT_WEBAPPS}/
                """
            }
        }

        stage('Restart Tomcat') {
            steps {
                sh """
                    echo "Restarting Tomcat..."
                    ssh -o StrictHostKeyChecking=no ubuntu@98.81.60.180 \
    /home/ubuntu/apache-tomcat-9.0.109/bin/shutdown.sh && \
    /home/ubuntu/apache-tomcat-9.0.109/bin/startup.sh

                """
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed!'
        }
    }
}
