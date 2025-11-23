pipeline {
  agent any

  parameters {
    choice choices: ['javaapp-pipeline', 'javaapp-standalone', 'javaapp-tomcat'], name: 'folder'
  }

  stages {
    stage('Checkout') {
      steps {
        git 'https://github.com/prashanth0996/jenkins.git'
      }
    }

    stage('Test') {
      steps {
        sh """
          echo "The test is started"
          cd ${folder}
          mvn clean test
        """
      }
    }

    stage('Build') {
      steps {
        sh """
          echo "The build is started"
          cd ${folder}
          mvn clean package -DskipTests
        """
      }
    }

stage('Code Analysis') {

                        steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        cd ${folder} && pwd
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=java \
                        -Dsonar.projectName=java 
                    '''
                }
            }
        }


stage('Upload to Artifactory') {
    when {
        branch 'main'
          }
              steps {
                rtUpload(
                    serverId: 'Jfrog',
                    spec: '''{
                        "files": [{
                            "pattern": "${folder}/target/*.*ar",
                            "target": "Maven/2.${BUILD_NUMBER}/"
                        }]
                    }'''
                )
            }
        }  


stage ('Manual Approval' ) {
	when {
        branch 'main'
          }
     options{ 
        timeout( time: 1 , unit: 'MINUTES') 
      }
	steps{
       input 'Approval for the Deploy'
	}
	}
   

stage ('Deploy') {
 when {
        branch 'main'
          }
	steps {
        sh """
        echo "The Deploy Started"
        cd ${folder}/target
	sudo cp *.*ar /opt/tomcat/webapps/
	echo "Deployed completed"
          """
	}
	}



  }

post {
	when {
        branch 'main'
          }
    always {
      echo "Clean the work space after post build"
      cleanWs()
    }
  }	
}
