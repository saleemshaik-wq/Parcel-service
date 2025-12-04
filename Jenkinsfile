pipeline { 
    agent any

    tools { 
        jdk 'java17'
        maven 'maven3'
    } 

    stages {
        stage('Checkout Code') {
            steps { 
                checkout scm
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                echo "Artifact archived successfully."
            }
        }

        stage('Run Application & Validate') {
            steps {
                script {
                    echo "Starting Spring Boot App..."

                    // Run app in background
                    sh 'nohup java -jar target/*.jar > app.log 2>&1 & echo $! > app.pid'

                    sleep 10

                    echo "Checking if app started..."

                    def status = sh(script: 'curl --write-out "%{http_code}" --silent --output /dev/null http://localhost:8080', 
                                     returnStdout: true).trim()

                    if (status != "200") {
                        error "App failed to start! HTTP Status: ${status}"
                    } else {
                        echo "App started successfully ✔"
                    }
                }
            }
        }

        stage('Wait 5 minutes') {
            steps {
                echo 'Keeping app running for 5 minutes...'
                sleep(time: 5, unit: 'MINUTES')
            }
        }

        stage('Stop Application') {
            steps {
                script {
                    echo "Stopping app..."
                    sh 'kill $(cat app.pid) || true'
                }
            }
        }
    }

    post {
        always {
            echo "Post cleanup..."
            sh 'pkill -f "java -jar" || true'
        }
    }
}
