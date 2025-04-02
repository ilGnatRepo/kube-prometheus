pipeline {
    agent any
    environment {
        DOCKER_HOST = 'tcp://10.110.0.6:2375'
    }
    stages {
        // stage("deploy monitor package") {
        //     steps {
        //         script {
        //             docker.withServer("${DOCKER_HOST}") {
        //                 docker.image('alpine/ansible:latest').inside('-v /home/ubuntu/.ssh/known_hosts:/root/.ssh/known_hosts') {
        //                     ansiColor('xterm') {
        //                         dir('ansible') {
        //                             ansiblePlaybook(
        //                                 playbook: 'playbook.yml',
        //                                 inventory: 'inventory.yml',
        //                                 credentialsId: '28488188-7eef-453b-bfb5-7b230ebb8688',
        //                                 colorized: true
        //                             )
        //                         }
        //                     }                        
        //                 }
        //             }
        //         }
        //     }
        // }
        stage("check kubectl remote config") {
            steps {
                script {
                    docker.withServer("${DOCKER_HOST}") {
                        withCredentials([file(credentialsId: '0ac81849-bb67-4d9d-b342-769e49277d08', variable: 'KUBECONFIG')]) {
                            docker.image('alpine/k8s:1.32.2').inside('-v ${KUBECONFIG}:/root/.kube/config --entrypoint=""') {
                                sh """
                                    # Install kube-prometheus
                                    kubectl apply --server-side -f manifests/setup --request-timeout=10m
                                    kubectl wait --for condition=Established --all CustomResourceDefinition --namespace=monitoring
                                    kubectl apply -f manifests/
                                    # Uninstall kube-prometheus
                                    # kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
                                    kubectl get pods -n monitoring -o wide
                                """
                            }
                        }
                    }
                }
            }
        }
    }
}