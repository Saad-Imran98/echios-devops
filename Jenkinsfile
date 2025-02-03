pipeline {
    agent any
    tools {
        maven "m3"
    }

    stages {

        stage('git check') {
            steps {
                sh "ls -l"
                sh "rm -rf /root/.m2/repository/*"
            }
        }

        stage('Build') {
            steps {
                dir('calculator') {
                sh 'cat src/main/java/com/echios/App.java '
                sh 'mvn clean install'
                }
            }
        }
    }
}

