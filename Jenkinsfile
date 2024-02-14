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
        stage("build") {    
            steps {
                echo "---------build started---------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'   //mvn=maven command nad maven there're 7 lifecyles,we can use any lifecyles. Here, goals are clean and deploy.
                echo "---------build completed---------" //when deploy command is mentioned, it'll execute unit test cases as well to avoid that we're using command "-Dmaven.test.skip=true"
            }
        }

        stage("test") { //for unit test 
            steps {
                echo "---------unit test started---------"
                sh 'mvn surefire-report:report'     //this command is to run unit test cases separatly.
                echo "---------unit test completed---------"
            }
        }

        stage('SonarQube analysis') { //this stage is a groovy scr //sonarqube scanner
        environment{
            scannerHome = tool 'mahi-sonar-scanner' //sonnar scannar location i.e name in our jenkins tools
        }
            steps{
                withSonarQubeEnv('mahi-sonarqube-server') { //sonar server name in our jenkins system
                sh "${scannerHome}/bin/sonar-scanner"
                }        
            }
        }
        
        stage("Quality Gate"){
            steps {
                script {  //this is a groovy script so coverted to declarivtive script by adding steps and script lines   //googlesearch:sonar stage for jenkinsfile--2nd link--Scripted pipeline example 
                    timeout(time: 1, unit: 'HOURS') {  //here it is 1hr // Just in case something goes wrong, pipeline will be killed after a timeout
                        def qg = waitForQualityGate() //passed or failed from sonar QG // Reuse taskId previously collected by withSonarQubeEnv
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }    
                }
            }    
        }   
    }
}    
