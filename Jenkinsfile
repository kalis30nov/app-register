pipeline {
    agent {label 'Jenkins-Agent'}
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    stages{
        stage("Cleanup Workspace"){
            steps{
                cleanWs()
            }
        }
    }
    stages{
        stage("Checkout from SCM"){
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/kalis30nov/app-register'
            }
        }
    }
    stages{
        stage("Build Application"){
            steps{
                sh "mvn clean package"
            }
        }
    }
    stages{
        stage("Test Application"){
            steps{
                sh "mvn test"
            }
        }
    }
}
