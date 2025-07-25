pipeline {
    agent any
    
    tools {
        maven 'Maven 3.9.11'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                echo "Build number: ${env.BUILD_NUMBER}"
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building Maven project...'
                dir('complete') {
                    sh 'mvn clean install'
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'complete/target/*.jar', fingerprint: true
                    // Публікація результатів тестів
                    junit testResults: 'complete/target/surefire-reports/*.xml', allowEmptyResults: true
                }
            }
        }
        
        stage('Deploy to EC2') {
            steps {
                echo 'Deploying to EC2...'
                // Використовуємо Publish Over SSH plugin
                publishOverSSH([
                    publishers: [
                        [
                            configName: 'test-pet',
                            transfers: [
                                [
                                    sourceFiles: 'complete/target/*.jar',
                                    removePrefix: 'complete/target/',
                                    remoteDirectory: '/opt/spring-app/',
                                    execCommand: '''
                                        cd /opt/spring-app && \
                                        ls -la *.jar && \
                                        ./start-app.sh && \
                                        sleep 15 && \
                                        curl -f http://localhost:8080 && \
                                        echo "Deployment completed successfully"
                                    '''
                                ]
                            ]
                        ]
                    ]
                ])
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline completed'
        }
        success {
            echo '🎉 Pipeline succeeded!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
