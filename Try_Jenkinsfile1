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
          cd ${params.folder}
          mvn clean test
        """
      }
    }

    stage('Build') {
      when {
        branch 'main'
      }
      steps {
        sh """
          echo "The build is started"
          cd ${params.folder}
          mvn clean package -DskipTests
        """
      }
    }

    stage('Deploy') {
      when {
        branch 'main'
      }
      steps {
        sh """
          echo "The Deploy is started"
          cd ${params.folder}/target
          JENKINS_NODE_COOKIE=dontKillMe nohup java -jar java-sample-21-1.0.0.jar > out.log 2>&1 &
          sudo netstat -tulnp | grep 5000 || echo "Deployment check completed"
        """
      }
    }
  }
}
