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
                withCredentials([string(credentialsId: env.GDS_PAAS_CONFIG, variable: 'paas_config_raw')]) {
                  paas_config = readJSON text: paas_config_raw
                }
                if (!config.PAAS_REGION) {
                  config.PAAS_REGION = paas_config.default
                }
                paas_region = paas_config.regions."${config.PAAS_REGION}"
                echo "\u001B[32mINFO: Setting PaaS region to ${paas_region.name}.\u001B[m"

                withCredentials([usernamePassword(credentialsId: paas_region.credential, passwordVariable: 'gds_pass', usernameVariable: 'gds_user')]) {
                  sh """
                    cf api ${paas_region.api}
                    cf auth ${gds_user} ${gds_pass}
                  """
                }

                gds_app = params.cf_app.split("/")
                sh "cf target -o ${gds_app[0]} -s ${gds_app[1]}"
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
