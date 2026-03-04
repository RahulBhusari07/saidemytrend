pipeline {
    agent any

    environment {
        PATH = "/opt/maven/bin:$PATH"
        REGISTRY = "https://trialg54ldq.jfrog.io"
    }

    stages {

        stage('Build') {
            steps {
                echo "------------ Build Started ------------"
                sh 'mvn clean package -Dmaven.test.skip=true'
                echo "------------ Build Completed ------------"
            }
        }

        stage('Test') {
            steps {
                echo "------------ Unit Test Started ------------"
                sh 'mvn surefire-report:report'
                echo "------------ Unit Test Completed ------------"
            }
        }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'saidemy-sonar-scanner'
            }
            steps {
                echo "------------ SonarQube Analysis Started ------------"
                withSonarQubeEnv('saidemy-sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()

                        if (qg.status != 'OK') {
                            echo "⚠ WARNING: Quality Gate Failed (${qg.status}) but continuing pipeline"
                            currentBuild.result = 'UNSTABLE'
                        } else {
                            echo "Quality Gate Passed"
                        }
                    }
                }
            }
        }

        stage('Jar Publish') {
            steps {
                script {

                    echo "------------ JAR Publish Started ------------"

                    def server = Artifactory.newServer(
                        url: "${REGISTRY}/artifactory",
                        credentialsId: "artifact-cred"
                    )

                    def properties = "buildid=${env.BUILD_ID};commitid=${env.GIT_COMMIT}"

                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/*.jar",
                                "target": "sai-libs-release-local/",
                                "flat": "true",
                                "props": "${properties}"
                            }
                        ]
                    }"""

                    def buildInfo = server.upload(uploadSpec)

                    buildInfo.env.collect()

                    server.publishBuildInfo(buildInfo)

                    echo "------------ JAR Publish Completed ------------"
                }
            }
        }
    }
}
