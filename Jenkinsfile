pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: docker
      image: docker:20.10.7
      command: ['sleep']
      args: ['99d']
      tty: true
      securityContext:
        runAsUser: 0
      volumeMounts:
        - name: docker-socket
          mountPath: /var/run/docker.sock
  volumes:
    - name: docker-socket
      hostPath:
        path: /var/run/docker.sock
'''
            label 'docker-agent-pod-root'
        }
    }
    stages {
        stage('Build Docker Image') {
            steps {
                container('docker') {
                    script {
                        echo "Начинаем сборку Docker-образа..."
                        sh 'docker build -t my-devops-app:latest .'
                        echo "Сборка Docker-образа завершена."
                    }
                }
            }
        }
        
        // --- НАШ НОВЫЙ ЭТАП ---
        stage('Push to Registry') {
            steps {
                container('docker') {
                    script {
                        // Переменная DOCKER_HUB_USERNAME должна соответствовать вашему логину на Docker Hub
                        def DOCKER_HUB_USERNAME = "procherar" // <-- ЗАМЕНИТЕ НА СВОЙ ЛОГИН
                        def IMAGE_NAME = "${DOCKER_HUB_USERNAME}/my-devops-app:latest"

                        echo "Перетегируем образ в ${IMAGE_NAME}"
                        sh "docker tag my-devops-app:latest ${IMAGE_NAME}"
                        
                        echo "Отправляем образ в Docker Hub..."
                        // Используем хранилище credentials, которое мы создали
                        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            sh "docker login -u ${env.DOCKER_USER} -p ${env.DOCKER_PASS}"
                            sh "docker push ${IMAGE_NAME}"
                            sh "docker logout"
                        }
                        echo "Отправка образа завершена."
                    }
                }
            }
        }
    }
}
