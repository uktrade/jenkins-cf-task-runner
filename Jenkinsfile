pipeline {

  agent {
    node {
      label 'docker.ci.uktrade.io'
    }
  }

  parameters {
    string(defaultValue: '', description:'Please enter app: ', name: 'cf_app')
    string(defaultValue: '', description:'Please enter task name: ', name: 'task_name')
    string(defaultValue: '', description:'Please enter task command: ', name: 'task_cmd')
    string(defaultValue: '1024M', description:'Please enter task mem: ', name: 'task_mem')
    string(defaultValue: '1G', description:'Please enter task disk: ', name: 'task_disk')
  }

  stages {

    stage('Init') {
      steps {
        script {
          timestamps {
            ansiColor('xterm') {
              deployer = docker.image("ukti/deployer:${env.GIT_BRANCH.split("/")[1]}")
              deployer.pull()
            }
          }
        }
      }
    }

    stage('Task') {
      steps {
        script {
          timestamps {
            ansiColor('xterm') {
              deployer.inside {
                gds_app = params.cf_app.split("/")
                withCredentials([usernamePassword(credentialsId: env.GDS_PAAS_CREDENTIAL, passwordVariable: 'gds_pass', usernameVariable: 'gds_user')]) {
                  sh """
                    cf login -a ${env.GDS_PAAS} -u ${gds_user} -p ${gds_pass} -o ${gds_app[0]} -s ${gds_app[1]}
                    cf target -o ${gds_app[0]} -s ${gds_app[1]}
                  """
                  sh """
                    cf run-task ${gds_app[2]} '${env.task_cmd}' --name ${task_name} -k ${env.task_disk} -m ${env.task_disk}
                  """
                }
              }
            }
          }
        }
      }
    }

  }
}
