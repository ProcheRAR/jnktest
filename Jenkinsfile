pipeline {
    agent {
        // Мы говорим Jenkins, что хотим определить агента с помощью Kubernetes
        kubernetes {
            // Вместо использования отдельных опций, мы предоставляем полную YAML-спецификацию
            // для пода, который Jenkins создаст для нашей сборки.
            // Тройные кавычки ''' позволяют вставить многострочный текст.
            yaml '''
apiVersion: v1
kind: Pod
spec:
  # Определяем контейнеры, которые будут в нашем поде-агенте
  containers:
    # Первый (и основной для работы) контейнер, который содержит Docker CLI
    - name: docker
      image: docker:20.10.7
      # Команда, которая не дает контейнеру завершиться сразу после старта
      command: ['sleep']
      args: ['99d']
      tty: true
      # МОНТИРУЕМ том внутрь этого контейнера
      volumeMounts:
        - name: docker-socket
          mountPath: /var/run/docker.sock

  # ОПРЕДЕЛЯЕМ тома на уровне всего пода
  volumes:
    # Создаем том с именем 'docker-socket', который ссылается на путь на хост-машине
    - name: docker-socket
      hostPath:
        path: /var/run/docker.sock
'''
            // Мы можем дополнительно указать метку для этого агента
            label 'docker-agent-pod'
        }
    }
    stages {
        stage('Build Docker Image') {
            steps {
                // ВАЖНО: Мы должны явно указать, в каком из контейнеров пода выполнять команды.
                // В нашем случае это контейнер с именем 'docker'.
                container('docker') {
                    script {
                        echo "Начинаем сборку Docker-образа в YAML-агенте..."
                        // Добавим проверку версии для уверенности
                        sh 'docker --version'
                        // Теперь эта команда будет выполнена в контейнере, где есть Docker и доступ к сокету
                        sh 'docker build -t my-devops-app:latest .'
                        echo "Сборка Docker-образа завершена."
                    }
                }
            }
        }
    }
}
