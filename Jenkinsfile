pipeline {
 
  agent {
    node {
      label 'builder'
      customWorkspace '/home/jenkins/DockerGrafanaInfluxKit'
    }
  }

  environment {
    INFLUXDB_VOLUME_NAME = "dockergrafanainfluxkit_influxdb_data"
    GRAFANA_VOLUME_NAME = "dockergrafanainfluxkit_grafana_data"
  }
 
  options {
    timestamps()
    ansiColor('xterm')
    disableConcurrentBuilds()
    buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '3', daysToKeepStr: '', numToKeepStr: '20'))
    timeout(30)
  }
 
  stages {
    stage('Checkout') {
      steps {
        script {
          sh 'sudo rm -rf * || true'
          checkout scm
          currentBuild.description = "${env.NODE_NAME}"
        }
      }
    }
 
    stage('Stop') {
      steps {
        script {
          sh """
            docker-compose down || true
            sleep 10
          """
        }
      }
    }
 
    stage('Backup') {
      steps {
        script {
 
          sh """
            DATE=`date '+%d%m%Y-%H%M%S'`
            
            docker run --rm -t -v ${INFLUXDB_VOLUME_NAME}:/volume -v \$(pwd):/backup debian tar czvf /backup/${INFLUXDB_VOLUME_NAME}_\${DATE}.tar.gz /volume/
            docker run --rm -t -v ${GRAFANA_VOLUME_NAME}:/volume -v \$(pwd):/backup debian tar czvf /backup/${GRAFANA_VOLUME_NAME}_\${DATE}.tar.gz /volume/
          """
          archiveArtifacts "*.tar.gz"
        }
      }
    }
 
    stage('Setup') {
      steps {
        script {
          sh """
            docker-compose build
          """
        }
      }
    }
 
    stage('Deploy') {
      steps {
        script {
          sh """
            nohup docker-compose up -d
          """
        }
      }
    }
  }
 
  post {
    always {
      step([$class: 'Mailer', notifyEveryUnstableBuild: true, recipients: "${emailextrecipients([[$class: 'RequesterRecipientProvider']])}", sendToIndividuals: true])
    }
  }
}