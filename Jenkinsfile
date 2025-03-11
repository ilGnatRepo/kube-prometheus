pipeline {
    agent any
    stages {
        stage("deploy monitor package") {
            steps {
                script {
                    docker.image('alpine/ansible:latest').inside('-v /home/ubuntu/.ssh/known_hosts:/root/.ssh/known_hosts') {
                        ansiColor('xterm') {
                            dir('ansible') {
                                ansiblePlaybook(
                                    playbook: 'playbook.yml',
                                    inventory: 'inventory.yml',
                                    credentialsId: '40790f20-323f-45ef-be89-2f74abd72ef8',
                                    colorized: true
                                )
                            }
                        }                        
                    }
                }
            }
        }
        stage("check kubectl remote config") {
            steps {
                script {
                    withCredentials([file(credentialsId: '899b1b6e-c500-43de-8c22-6522375b44ee', variable: 'KUBECONFIG')]) {
                        docker.image('alpine/k8s:1.32.2').inside('-v ${KUBECONFIG}:/root/.kube/config --entrypoint=""') {
                            sh """
                                kubectl get pods -n monitoring -o wide
                                kubectl apply --server-side -f manifests/setup
                                kubectl wait \
                                        --for conidition=Established \ 
                                        --all CustomResourceDefinition \
                                        --namespace=monitoring
                                kubectl apply -f manifests/
                            """
                        }
                    }
                }
            }
        }
    }
}