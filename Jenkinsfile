pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "tharik007/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'sample-edureka-password') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-credentials-id', variable: 'KUBECONFIG')]) {
                sh """
                    export KUBECONFIG=\$KUBECONFIG
                    kubectl apply -f train-schedule-kube-canary.yml.yaml
                """
                // kubernetesDeploy(
                //     kubeconfigId: 'kubeconfig',
                //     configs: 'train-schedule-kube-canary.yml',
                //     enableConfigSubstitution: true
                // )
                // script {
                    // def kubeconfigPath = writeKubeconfigToFile(KUBECONFIG)
                    // def kubectl = tool name: 'kubectl', type: 'ToolType'
                    
                    // sh "cat ${kubeconfigPath}" // Just to verify the kubeconfig content.
                    // def kubeconfig = credentials('kubeconfig-credentials-id')
                    // sh "kubectl --kubeconfig=${kubeconfig} apply -f train-schedule-kube-canary.yml"
                    // sh "sudo kubectl apply -f train-schedule-kube-canary.yml"
                    // Replace 'your-kubernetes-manifest.yaml' with the actual path to your Kubernetes manifest YAML file.
                // }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
