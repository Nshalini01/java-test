pipeline {
    agent any

    environment {
        function_name = 'mylambda'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building'
                sh 'mvn package'
            }
        }
stage('SonarQube analysis') {
            when {
                anyOf {
                    branch 'main'
                }
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    echo 'Scanning'
                    sh 'mvn sonar:sonar'
                }
            }
        }
        
        stage("Quality Gate") {
            steps {
                script {
                    try {
                        timeout(time: 1, unit: 'MINUTES') {
                            def qualityGate = waitForQualityGate abortPipeline: true
                            echo "Quality Gate status is ${qualityGate.status}"
                            echo "Quality Gate details: ${qualityGate}"
                        }
                    } catch (Exception e) {
                        echo "Quality Gate failed: ${e.getMessage()}"
                    }
                }
            }
        }
        stage('Push') {
            steps {
                echo 'Push'

                sh "aws s3 cp target/sample-1.0.3.jar s3://nalluri-1"
            }
        }

        stage('Deploy') {
            steps {
                echo 'Build'

                sh "aws lambda update-function-code --function-name $function_name --region us-east-2 --s3-bucket nalluri-1 --s3-key sample-1.0.3.jar"
            }
        }
    }
}
