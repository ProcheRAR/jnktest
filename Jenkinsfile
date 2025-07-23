pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
    # Контейнер №1: Docker CLI (остается без изменений)
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

    # --- НАШ НОВЫЙ КОНТЕЙНЕР ДЛЯ KUBECTL ---
    # Контейнер №2: kubectl
    - name: kubectl
      image: bitnami/kubectl:1.28
      command: ['sleep']
      args: ['99d']
      tty: true

  volumes:
    - name: docker-socket
      hostPath:
        path: /var/run/docker.sock
'''
            label 'ci-cd-agent'
        }
    }
    stages {
        stage('Build & Push') {
            // Объединим два этапа для эффективности
            steps {
                container('docker') {
                    script {
                        def DOCKER_HUB_USERNAME = "procherar" // <-- ЗАМЕНИТЕ НА СВОЙ ЛОГИН
                        def IMAGE_NAME = "${DOCKER_HUB_USERNAME}/my-devops-app:latest"

                        echo "1. Сборка Docker-образа..."
                        sh "docker build -t ${IMAGE_NAME} ."
                        
                        echo "2. Отправка образа в Docker Hub..."
                        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            sh "docker login -u ${env.DOCKER_USER} -p ${env.DOCKER_PASS}"
                            sh "docker push ${IMAGE_NAME}"
                            sh "docker logout"
                        }
                    }
                }
            }
        }

        // --- НАШ ФИНАЛЬНЫЙ ЭТАП РАЗВЕРТЫВАНИЯ ---
        stage('Deploy to k3s') {
            steps {
                // Выполняем команды в контейнере с kubectl
                container('kubectl') {
                    script {
                        echo "Развертываем приложение в кластере k3s..."
                        sh 'kubectl apply -f k8s/deployment.yaml'

                        echo "Ожидаем готовности развертывания..."
                        sh 'kubectl rollout status deployment/my-devops-app-deployment'
                        echo "Развертывание успешно завершено!"
                    }
                }
            }
        }
    }
}
