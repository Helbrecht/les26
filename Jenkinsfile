pipeline {
    agent any

    tools {
        maven 'Maven'  
    }
	
	environment {
        SONAR_TOKEN = credentials('sonar-token')  
    }
	
    stages {
        stage('Detect Changed Apps') {
            steps {
                script {
                    def changedFiles = bat(script: "git diff --name-only HEAD~1 || echo ''", returnStdout: true).trim().split('\n')

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

		stage('SonarQube Analysis') {
            steps {
				retry(3) {
				sleep(time: 60, unit: 'SECONDS')
                script {
                    withSonarQubeEnv('SonarQube') {
                        dir('HelloWorld') {
                            bat 'mvn clean verify sonar:sonar -Dsonar.projectKey=my-java-apps -Dsonar.projectName="HelloWorld"'
                        }
                        dir('HelloJenkins') {
                            bat 'mvn clean verify sonar:sonar -Dsonar.projectKey=my-java-apps -Dsonar.projectName="HelloJenkins"'
                        }
                        dir('HelloDevops') {
                            bat 'mvn clean verify sonar:sonar -Dsonar.projectKey=my-java-apps -Dsonar.projectName="HelloDevops"'
                        }
                    }
					}
                }
            }
        }
		
		stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('SonarQube') {
            script {
                def appsToAnalyze = env.CHANGED_APPS.split(',')
                appsToAnalyze.each { app ->
                    dir(app) {
                        def sonarResult = bat(script: 'mvn clean verify sonar:sonar -Dsonar.projectKey=my-java-apps -Dsonar.projectName="${app}"', returnStdout: true)
                        echo sonarResult
                       
                        def taskUrl = (sonarResult =~ /http:\/\/localhost:9000\/api\/ce\/task\\?id=([A-Za-z0-9]+)/)
                        if (taskUrl) {
                            echo "Ссылка на задачу SonarQube: http://localhost:9000/dashboard?id=my-java-apps#sonarqube-task=${taskUrl[0][1]}"
                        }
                    }
                }
            }
        }
    }
}
        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 20, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
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
                            bat 'mvn clean package'
                            echo "=== Запуск HelloWorld ==="
                            bat 'java -jar target/hello-app-1.0-SNAPSHOT.jar'
                        }
                    }
                }
                stage('HelloJenkins') {
                    when { expression { env.CHANGED_APPS.contains('HelloJenkins') } }
                    steps {
                        dir('HelloJenkins') {
                            echo "=== Сборка HelloJenkins ==="
                            bat 'mvn clean package'
                            echo "=== Запуск HelloJenkins ==="
                            bat 'java -jar target/hello-app-1.0-SNAPSHOT.jar'
                        }
                    }
                }
                stage('HelloDevops') {
                    when { expression { env.CHANGED_APPS.contains('HelloDevops') } }
                    steps {
                        dir('HelloDevops') {
                            echo "=== Сборка HelloDevops ==="
                            bat 'mvn clean package'
                            echo "=== Запуск HelloDevops ==="
                            bat 'java -jar target/hello-app-1.0-SNAPSHOT.jar'
                        }
                    }
                }
            }
        }
    }
}
