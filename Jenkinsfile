pipeline {
    environment {
        dockerImageName = "jeonghaerim/simple-echo"
    }
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: jnlp
                image: sheayun/jnlp-agent-sample
                env:
                - name: DOCKER_HOST
                  value: "tcp://localhost:2375"
              - name: dind
                image: docker:latest
                command:
                - /usr/local/bin/dockerd-entrypoint.sh
                env:
                - name: DOCKER_TLS_CERTDIR
                  value: ""
                securityContext:
                  privileged: true
            '''
        }
    }
    stages {
        stage("git scm update") {
            steps {
                checkout scm
            }
        }
        stage("docker build && push") {
            steps {
                script {
                    // Docker 이미지 빌드
                    dockerImage = docker.build dockerImageName
                    
                    // Docker 이미지를 Docker Hub에 푸시
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                        dockerImage.push("latest")
                    }
                }
            }
        }
        stage("deploy application on kubernetes cluster") {
            steps {
                script {
                    // 네임스페이스가 없는 경우 생성
                    sh 'kubectl create namespace default || true'
                    withKubeConfig([credentialsId: "KUBECONFIG", serverUrl: "https://kubernetes.default", namespace: "default"]){
                        sh '''
                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml
                        '''
                    }
                }
            }
        }
    }
    post {
        always {
            sh "docker logout"   
        }
    }
}
