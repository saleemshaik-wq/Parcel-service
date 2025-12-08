pipeline {
    agent { label 'slave2' }

    tools {
        jdk 'java17'
        maven 'maven3'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Run App') {
            steps {
                echo "Starting App..."

                sh '''
                    nohup java -jar target/*.jar > app.log 2>&1 &
                    echo $! > app.pid
                '''

                sleep 15
            }
        }

        stage('Validate App') {
            steps {
                echo "Checking app..."

                script {
                    def code = sh(
                        script: 'curl -s -o /dev/null -w "%{http_code}" http://localhost:8081',
                        returnStdout: true
                    ).trim()

                    if (code != "200") {
                        error "App failed to start! Status: ${code}"
                    } else {
                        echo "App is running ✔"
                    }
                }
            }
        }

        stage('Stop App') {
            steps {
                echo "Stopping App..."

                sh '''
                    if [ -f app.pid ]; then
                        kill $(cat app.pid) || true
                    fi
                '''
            }
        }
    }

    post {
        always {
            echo "Cleanup"
            sh '''
                if [ -f app.pid ]; then
                    kill $(cat app.pid) || true
                fi
            '''
        }
    }
}
