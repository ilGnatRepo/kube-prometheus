pipeline {
    agent any
    stages {
        stage("deploy monitor package") {
            steps {
                script {
                    docker.image('alpine/ansible:latest').inside('-v /root/.ssh/known_hosts:/root/.ssh/known_hosts') {
                        ansiColor('xterm') {
                            dir('ansible') {
                                ansiblePlaybook(
                                    playbook: 'playbook.yml',
                                    inventory: 'inventory.yml',
                                    credentialsId: 'b42216b6-0415-41da-bdca-bb76291352b9',
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
                    withCredentials([file(credentialsId: '9d02a0a5-e1d0-45b8-a842-79520936eccd', variable: 'KUBECONFIG')]) {
                        docker.image('alpine/k8s:1.32.2').inside('-v ${KUBECONFIG}:/root/.kube/config --entrypoint=""') {
                            sh """
                                # Install kube-prometheus
                                kubectl apply --server-side -f manifests/setup
                                kubectl wait --for condition=Established --all CustomResourceDefinition --namespace=monitoring
                                kubectl apply -f manifests/
                                kubectl apply -f monitoring-ingress.yml
                                # Uninstall kube-prometheus
                                # kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
                            """
                        }
                    }
                }
            }
        }
    }
}