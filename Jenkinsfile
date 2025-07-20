pipeline {
    // Определяем агента НЕ на верхнем уровне, а внутри этапа,
    // или используем специальный синтаксис для описания окружения.
    // Этот вариант самый правильный для Kubernetes.
    agent {
        // Указываем, что хотим использовать Kubernetes для запуска агента
        kubernetes {
            // Имя пода, которое будет отображаться в Kubernetes.
            label 'docker-agent'
            // Определяем контейнер, в котором будут выполняться шаги
            containers {
                // Имя контейнера и образ, который нужно использовать.
                // Мы берем официальный образ Docker, в котором уже есть Docker CLI.
                containerTemplate(name: 'docker', image: 'docker:20.10.7', command: 'sleep', args: '99d', ttyEnabled: true)
            }
            // --- САМАЯ ВАЖНАЯ ЧАСТЬ ---
            // Пробрасываем сокет Docker с хост-машины внутрь этого пода-агента.
            // То же самое, что мы делали в values.yaml, но теперь для агента.
            volumes {
                hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
            }
        }
    }
    stages {
        stage('Build Docker Image') {
            steps {
                // Указываем, что шаги внутри этого блока должны выполняться
                // в контейнере с именем 'docker', который мы определили выше.
                container('docker') {
                    script {
                        echo "Начинаем сборку Docker-образа в кастомном агенте..."
                        // Эта команда теперь будет выполняться в контейнере, где есть Docker!
                        sh 'docker build -t my-devops-app:latest .'
                        echo "Сборка Docker-образа завершена."
                    }
                }
            }
        }
        // Здесь мы можем добавить следующий этап
        // stage('Push to Registry'){ ... }
    }
}
