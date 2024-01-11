pipeline {
  agent any
  tools {
    maven 'MAVEN3'
    jdk 'Oraclejdk11'
  }

  stages {
    stage('Cloning') {
      steps {
        git branch: 'main', url: 'https://github.com/riyanuddin17/vprofile-project.git'
      }
    }

    stage('BUILD') {
      steps {
        sh 'mvn clean install -DskipTests'
      }
      post {
        success {
          echo 'Now Archiving...'
          archiveArtifacts artifacts: '**/target/*.war'
        }
      }
    }

    stage('UNIT TEST') {
      steps {
        sh 'mvn test'
      }
    }

    stage('INTEGRATION TEST') {
      steps {
        sh 'mvn verify -DskipUnitTests'
      }
    }

    stage('CODE ANALYSIS WITH CHECKSTYLE') {
      steps {
        sh 'mvn checkstyle:checkstyle'
      }
      post {
        success {
          echo 'Generated Analysis Result'
        }
      }
    }

    stage('CODE ANALYSIS with SONARQUBE') {

      environment {
        scannerHome = tool 'sonar4.7'
      }

      steps {
        withSonarQubeEnv('sonar') {
          sh '''${scannerHome}/bin/sonar-scanner -X -Djavax.net.debug=all -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/classes \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
        }

        timeout(time: 10, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
    stage("Uploading Artifact to Nexus"){
        steps{
        nexusArtifactUploader(
            nexusVersion: 'nexus3',
            protocol: 'http',
            nexusUrl: '172.31.82.242:8081',
            groupId: 'QA',
            version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
            repository: 'vprofile-repo',
            credentialsId: 'nexuslogin',
            artifacts: [
                [artifactId: 'vproapp',
                classifier: '',
                 file: 'target/vprofile-v2.war',
                 type: 'war']
                ]
            )
        }
    }


    stage("Getting Down the previous Containers") {
      steps {
        echo "Deploying the container"

        sh "docker-compose -f /var/lib/jenkins/workspace/test1/docker-compose.yml down"

      }
    }
    stage("Getting Up the New Containers") {
      steps {
        echo "Deploying the container"

        sh "docker-compose -f /var/lib/jenkins/workspace/test1/docker-compose.yml up -d"

      }
    }

  }
}
