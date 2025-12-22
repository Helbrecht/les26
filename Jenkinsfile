pipeline {
    agent any

    stages {
        stage('Detect Changed Apps') {
            steps {
                script {
                   
                    def changedFiles = sh(script: "git diff --name-only HEAD~1 || echo ''", returnStdout: true).trim().split('\n')

                    def apps = ['HelloWorld', 'HelloJenkins', 'HelloDevops']
                    def changedApps = []

                    apps.each { app ->
                        if (changedFiles.any { it.startsWith("${app}/") }) {
                            changedApps << app
                        }
                    }

                   
                    if (changedApps.isEmpty()) {
                        changedApps = apps
                    }

                    env.CHANGED_APPS = changedApps.join(',')
                    echo "Изменённые приложения: ${env.CHANGED_APPS}"
                }
            }
        }

        stage('Build & Run') {
            parallel {
                stage('HelloWorld') {
                    when { expression { env.CHANGED_APPS.contains('HelloWorld') } }
                    steps {
                        dir('HelloWorld') {
                            echo "=== Сборка HelloWorld ==="
                            sh 'mvn clean package'
                            echo "=== Запуск HelloWorld ==="
                            sh 'java -jar target/hello-app-1.0-SNAPSHOT.jar'
                        }
                    }
                }
                stage('HelloJenkins') {
                    when { expression { env.CHANGED_APPS.contains('HelloJenkins') } }
                    steps {
                        dir('HelloJenkins') {
                            echo "=== Сборка HelloJenkins ==="
                            sh 'mvn clean package'
                            echo "=== Запуск HelloJenkins ==="
                            sh 'java -jar target/hello-app-1.0-SNAPSHOT.jar'
                        }
                    }
                }
                stage('HelloDevops') {
                    when { expression { env.CHANGED_APPS.contains('HelloDevops') } }
                    steps {
                        dir('HelloDevops') {
                            echo "=== Сборка HelloDevops ==="
                            sh 'mvn clean package'
                            echo "=== Запуск HelloDevops ==="
                            sh 'java -jar target/hello-app-1.0-SNAPSHOT.jar'
                        }
                    }
                }
            }
        }
    }
}