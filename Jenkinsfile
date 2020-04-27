pipeline {
        environment {
            registryCredential = "jupiter-dockerhub"
            dockerImage = ""
            registry = "teamjupitergmu/mattermost-docker"
        }
        agent any
        stages {
            stage('Build & Publish new Docker images') {
                steps {
                    echo 'Starting to build docker image DB'
                    script {
                        dir ('db') {
                            dockerImage = docker.build(registry + ":db-${currentBuild.number}")
                            docker.withRegistry("https://registry.hub.docker.com", registryCredential){
                            dockerImage.push()
                           }
                         }
                        }
                    echo 'Starting to build docker image APP'
                    script {
                        dir ('app') {
                            dockerImage = docker.build(registry + ":app-${currentBuild.number}")
                            docker.withRegistry("https://registry.hub.docker.com", registryCredential){
                            dockerImage.push()
                           }
                         }
                        }
                    echo 'Starting to build docker image WEB'
                    script {
                        dir ('web') {
                            dockerImage = docker.build(registry + ":web-${currentBuild.number}")
                            docker.withRegistry("https://registry.hub.docker.com", registryCredential){
                            dockerImage.push()
                           }
                         }
                        }
                	}
             	    }
            stage('Deploy') {
                steps {
                sshagent(credentials : ['ansible-admin']) {
                   echo "current build number: ${currentBuild.number}"
                   echo "previous build number: ${currentBuild.previousBuild.getNumber()}"
                    sh "ssh ansible-admin@ansible.boppatea.com sed -i 's/-${currentBuild.previousBuild.getNumber()}/-${currentBuild.number}/g' /opt/docker/ansible/k8s-mattermost-deployment.yml"
                    sh "ssh ansible-admin@ansible.boppatea.com ansible-playbook /opt/docker/ansible/k8s-mattermost-deployment.yml -u k8s-admin -i /opt/docker/ansible/hosts"
                 }
                }
            }
            stage('Cleanup') {
                steps {
                    sh "docker rmi '${registry}:db-${currentBuild.number}'"
                    sh "docker rmi '${registry}:app-${currentBuild.number}'"
                    sh "docker rmi '${registry}:web-${currentBuild.number}'"
                   }
                  }
         }
 }
