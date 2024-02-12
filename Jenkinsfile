pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    stages {
        stage('Clone-code') {
            steps {
                git branch: 'main', url: 'https://github.com/mahenderkb6-2/tweet-trend-12feb.git'
            }
        }
    }
}
