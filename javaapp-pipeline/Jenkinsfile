pipeline {
    agent {label 'agent'}

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/artisantek/jenkins.git'
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        dir('javaapp-pipeline') {
                            sh 'mvn clean test'
                        }
                    }
                }
                stage('Trivy Scan') {
                    steps {
                        dir('javaapp-pipeline') {
                            sh '''
                                wget -q https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl -O html.tpl
                                trivy fs --format template --template "@html.tpl" -o report.html .
                            '''
                        }
                    }
                }
            }
        }

        // stage('Sonar Analysis') {
        //     steps {
        //         dir('javaapp-pipeline') {
        //             withSonarQubeEnv('sonar') {
        //                 sh '''
        //                     mvn clean verify sonar:sonar \
        //                     -Dsonar.projectKey=java-app \
        //                     -Dsonar.projectName=java-app 
        //                 '''
        //             }
        //         }
        //     }
        // }

        stage('Build') {
            steps {
                dir('javaapp-pipeline') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Publish') {
            steps {
                script {
                    def GROUP_ID = sh(script: "mvn help:evaluate -Dexpression=project.groupId -q -DforceStdout -f javaapp-pipeline/pom.xml", returnStdout: true).trim()
                    def ARTIFACT_ID = sh(script: "mvn help:evaluate -Dexpression=project.artifactId -q -DforceStdout -f javaapp-pipeline/pom.xml", returnStdout: true).trim()
                    def VERSION = sh(script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout -f javaapp-pipeline/pom.xml", returnStdout: true).trim()
                    def GROUP_PATH = sh(script: "echo '${GROUP_ID}' | tr '.' '/'", returnStdout: true).trim()
                    def TARGET_PATH = "maven/${GROUP_PATH}/${ARTIFACT_ID}/${VERSION}/"
                    def JAR_PATH = "${ARTIFACT_ID}-${VERSION}.jar"
        
                    dir('javaapp-pipeline/target') {
                        rtUpload(
                            serverId: 'jfrog',
                            spec: """{
                                "files": [
                                    {
                                        "pattern": "${JAR_PATH}",
                                        "target": "${TARGET_PATH}"
                                    }
                                ]
                            }"""
                        )
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                dir('javaapp-pipeline/target') {
                    sh '''
                        if pgrep -f "java -jar java-sample-21-1.0.0.jar" > /dev/null; then
                            pkill -f "java -jar java-sample-21-1.0.0.jar"
                            echo "App was running and has been killed."
                        else
                            echo "App is not running."
                        fi
                        JENKINS_NODE_COOKIE=dontKillMe nohup java -jar java-sample-21-1.0.0.jar > app.log 2>&1 &
                    '''
                }
            }
        }

    }
    
    post {
      always {
        cleanWs()
      }
      failure {
        emailext attachLog: true, body: 'Pipeline Failed, Please check the logs', subject: 'Job Failed', to: 'artisantek.adithya@gmail.com'
      }
    }


}
