def imageName="marqsek/backend"
def dockerRegistry=""
def registryCredentials="dockerhub"
def dockerTag=""

pipeline {
    agent {
        label 'agent'
    }
 
    environment {
        PIP_BREAK_SYSTEM_PACKAGES = 1
        scannerHome = tool 'SonarQube'
    }
 
    stages {
        stage('Pobierz repo z gita') {
            steps {
                git branch: 'master', url: 'https://github.com/marqsek/Backend/'
            }
        }
 
        stage('testy jednostkowe') {
            steps {
                sh "pip3 install -r requirements.txt"
                sh "python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml"
            }
        }
 
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        } 
		
        stage('stworz obraz z dockera'){
            steps {
                script{
					dockerTag = "RC-${env.BUILD_ID}"
					applicationImage = docker.build("$imageName:$dockerTag")
                }
            }
        }
		
		stage ('publikacja obrazu') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                        applicationImage.push()
                        applicationImage.push('latest')
                    }
                }
            }
        }
    }
	
	post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
        success {
            build wait: false, job: 'App_of_apps', parameters: [string(name: 'backendDockerTag', value: "$dockerTag"), string(name: 'frontendDockerTag', value: '')]
        }
	}
}
