pipeline {
 agent any
  stages {
   stage('SCM Checkout'){
    steps {
       git credentialsId: 'git-creds', url: 'https://github.com/raikar/my-app.git'
    }
   }
   stage('Maven Test'){
    steps {
     sh "mvn test"
    }
   }
   stage('Maven Package and create CycloneDX BOM'){
    steps {
     sh "mvn clean install package org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom"
    }
   }
   stage('Dependency Check & Track') {
    steps {
          sh 'mvn org.owasp:dependency-check-maven:check -Dformat=XML -DdataDirectory=/usr/share/nvd -DautoUpdate=false'
          dependencyTrackPublisher artifact: 'target/bom.xml', artifactType: 'bom', failedNewCritical: 1, failedTotalCritical: 1, projectId: 'b15cc415-cf8f-455b-b567-5119f997ad29', synchronous: true, unstableNewCritical: 1, unstableTotalCritical: 1
     }
    }
   stage('SonarQube analysis') {
    steps {
    withSonarQubeEnv('Sonar-6') {
      sh 'mvn sonar:sonar'
     }
    } 
   }
   stage("SQ Quality Gate"){
    steps {
      echo "hello"
      }
    }
   stage('Build Docker Image'){
    steps {
     sh 'docker build -t raikar/my-app:2.0.1 .'
    }
   }
   stage('Push Docker Image'){
    steps {
     withCredentials([string(credentialsId: 'docker-password', variable: 'dockerHubPassword')]) {
     sh "docker login -u raikar -p ${dockerHubPassword}"
     }
    }
    steps {
     sh 'docker push raikar/my-app:2.0.1'
    }
   }
   stage('Docker Scan using Clair'){
    steps {
    sh "ssh -o StrictHostKeyChecking=no rajesh@192.168.254.4 ${DockerRun}"
    }
   }
   stage('Run Container on Dev Server'){
    steps {
     def DockerRun = 'docker run -p 8080:8080 -d --name rajesh-app-name raikar/my-app:2.0.1'
     sshagent(['rajeshPvtKeyforCentOS']) {
    sh "ssh -o StrictHostKeyChecking=no rajesh@192.168.254.3 ${DockerRun}"
     }
    }
   }
   stage ('Check using Gauntlt'){
    steps {
     def DockerRun = '/usr/local/bin/gauntlt-docker gauntlt-docker/examples/xss.attack'
     sshagent(['rajeshPvtKeyforCentOS']) {
      steps {
     sh "ssh -o StrictHostKeyChecking=no rajesh@192.168.254.3 ${DockerRun}"
      }
     }
    }
   }
   stage ('Check using ZAP'){
    steps {
     sh "ssh -o StrictHostKeyChecking=no rajesh@192.168.254.3 ${DockerRun}"
     }
    }
   stage ('Slack Notification'){
    steps {
     slackSend channel: 'jenkins-pipeline', 
     color: 'good', 
     message: "Build Completed with Security checks for - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", 
     teamDomain: 'theinfosec', 
     tokenCredentialId: 'slack-new-token'
     }
    }
   }
}
