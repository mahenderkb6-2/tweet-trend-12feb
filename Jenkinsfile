def registry = 'https://mahenderb14feb.jfrog.io/' //my jfrog url //for "Jar Publish" stage 
def imageName = 'mahenderb14feb.jfrog.io/mahi-docker/ttrend'    //for "Docker Build" stage //<jfrog artifact url>/<repository we're using to store our artifacts>/<name of artifact we want to give>
def version   = '2.1.5' //for "Docker Build" //artifact version mentioned in pom.xml at line ~13
pipeline {
    agent {
        node {
            label 'maven' //should be same label mentioned during jenkins node creation
        }
    }
environment {
    PATH = "/opt/apache-maven-3.9.6/bin:$PATH" //adding to my PATH variable with extra variable i.e. /opt/apache-maven-3.9.2/bin
}
    stages {
        stage("build") {    
            steps {
                echo "<--------------- build started --------------->"
                sh 'mvn clean deploy -Dmaven.test.skip=true'   //mvn=maven command and maven there're 7 lifecyles,we can use any lifecyles. Here, goals are clean and deploy.
                echo "<--------------- build completed --------------->" //when deploy command is mentioned, it'll execute unit test cases as well to avoid that we're using command "-Dmaven.test.skip=true"
            }
        }

        stage("test") { //for unit test 
            steps {
                echo "<---------------unit test started --------------->"
                sh 'mvn surefire-report:report'     //this command is to run unit test cases separatly.
                echo "<--------------- unit test completed --------------->"
            }
        }

        stage('SonarQube analysis') { //this stage is a groovy scr //sonarqube scanner
        environment{
            scannerHome = tool 'mahi-sonar-scanner' //sonar scannar location i.e name in our jenkins tools
        }
            steps{
                echo "<--------------- SonarQube analysis started --------------->"
                withSonarQubeEnv('mahi-sonarqube-server') { //sonar server name in our jenkins system
                sh "${scannerHome}/bin/sonar-scanner"
                echo "<--------------- SonarQube analysis completed --------------->"
                }        
            }
        }
        
        stage("Quality Gate"){ //to get status from sonarqube and  based on that it pass/fails the build
            steps {
                script {  //this is a groovy script so we coverted to declarivtive script by adding steps and script lines   //googlesearch:sonar stage for jenkinsfile--2nd link--Scripted pipeline example 
                    echo "<--------------- Quality Gate started --------------->"
                    timeout(time: 1, unit: 'HOURS') {  //here it is 1hr // Just in case something goes wrong, pipeline will be killed after a timeout
                        def qg = waitForQualityGate() //passed or failed from sonar QG // Reuse taskId previously collected by withSonarQubeEnv
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                    echo "<--------------- Quality Gate completed --------------->"    
                }
            }    
        }

        stage("Jar Publish") { //it stores the jar files in jfrog artifactory
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                    def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"mahi-jfrog-token"   //credentialsId= mahi-jfrog-token i.e. our Jgrog creds in jenkins
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"; //target means jfrog artifactory and it is in my jfrog web -->artifactory-->artifacts
                    def uploadSpec = """{          
                        "files": [
                            {
                                "pattern": "jarstaging/(*)",
                                "target": "mahi-libs-release/{1}",  
                                "flat": "false",
                                "props" : "${properties}",
                                "exclusions": [ "*.sha1", "*.md5"]
                            }
                        ]
                    }"""
                    def buildInfo = server.upload(uploadSpec) //uploading artifacts on to the artifact
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                    echo '<--------------- Jar Publish Ended --------------->'  
            
                }
            }   
        }
        
        stage("Docker Build") { //building image with copied files onto a container as a jar file
            steps {
                script {
                    echo '<--------------- Docker Build Started --------------->'
                    app = docker.build(imageName+":"+version)
                    echo '<--------------- Docker Build Ends --------------->'
                }
            }
        }

        stage ("Docker Publish"){ // publish docker artifact repository in jfrog artifactory
            steps {
                script {
                    echo '<--------------- Docker Publish Started --------------->'  
                    docker.withRegistry(registry, 'mahi-jfrog-token'){ //registry is a variable mentioned in 1st line for "Jar Publish" stage //"mahi-jfrog-token"=credentialsId i.e. our Jgrog creds in jenkins
                        app.push()
                    }    
                    echo '<--------------- Docker Publish Ended --------------->'  
                }
            }
        }

        stage(" Deploy ") {
            steps {
                script {
                    // sh './deploy.sh'
                    echo '<--------------- Helm Deploy Started --------------->'
                    sh 'helm install ttrend ttrend-0.1.0.tgz'
                    echo '<--------------- Helm deploy Ends --------------->'
                }
            }
        }  
    }
}            