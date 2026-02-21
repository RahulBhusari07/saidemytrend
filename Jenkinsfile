pipeline {
    agent any

    environment {
        PATH = "/opt/maven/bin:$PATH"
    }

    stages {

        stage('Build') {
            steps {
                echo "------------ build started ------------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "------------ build completed ------------"
            }
        }

        stage('Test') {
            steps {
                echo "------------ unit test started ------------"
                sh 'mvn surefire-report:report'
                echo "------------ unit test completed ------------"
            }
        }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'saidemy-sonar-scanner'
            }
            steps {
                withSonarQubeEnv('saidemy-sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "❌ Pipeline aborted due to Quality Gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }

    }
}
