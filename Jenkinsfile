pipeline {
    agent any
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Начинаем сборку Docker-образа..."
                    // Важно: эта команда будет выполняться ВНУТРИ пода Jenkins
                    sh 'docker build -t my-devops-app:latest .'
                    echo "Сборка Docker-образа завершена."
                }
            }
        }
        // Другие этапы (развертывание и т.д.) мы добавим позже
    }
}
