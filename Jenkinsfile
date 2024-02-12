pipeline {
    agent {
        node {
            label 'maven'
        }
    }
environment {
    PATH = "/opt/apache-maven-3.9.6/bin:$PATH" //adding to my PATH variable with extra variable i.e. /opt/apache-maven-3.9.2/bin
}
    stages {
        stage('Clone-code') {
            steps {
                sh 'mvn clean deploy' //mvn=maven command nad maven there're 7 lifecyles,we can use any lifecyles. Here, goals are clean and deploy.
            }
        }
    }
}
