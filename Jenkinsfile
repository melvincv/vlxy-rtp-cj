def imageName = 'chinar.jfrog.io/default-docker-local/valaxy-rtp'
def registry  = 'https://chinar.jfrog.io'
def version   = '1.0.0'
def app
pipeline {
    agent {
       node {
         label "slave"
      }
    }
    stages {
        stage('Build') {
            steps {
                echo '<--------------- Building --------------->'
                sh 'printenv'
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo '<------------- Build completed --------------->'
            }
        }
        stage('Unit Test') {
            steps {
                echo '<--------------- Unit Testing started  --------------->'
                sh 'mvn surefire-report:report'
                echo '<------------- Unit Testing stopped  --------------->'
            }
        }
        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'Sonarqubevj'
            }
            steps {
                echo '<--------------- Sonar Analysis Started --------------->'
                withSonarQubeEnv('sonarqubecj'){
                    sh "${scannerHome}/bin/sonar-scanner"
                }    
                echo '<--------------- Sonar Analysis Ends --------------->'
            }    
        }
        stage("Quality Gate") {
            steps {
                script {
                  echo '<--------------- Sonar Gate Analysis Started --------------->'
                    timeout(time: 1, unit: 'HOURS'){
                       def qg = waitForQualityGate()
                        if(qg.status !='OK') {
                            error "Pipeline failed due to quality gate failures: ${qg.status}"
                        }
                    }  
                  echo '<--------------- Sonar Gate Analysis Ends  --------------->'
                }
            }
        }
        stage("Docker Build") {
          steps {
            script {
               echo '<--------------- Docker Build Started --------------->'
               app = docker.build(imageName)
               echo '<--------------- Docker Build Ends --------------->'
            }
          }
        }
        stage("Jar Publish") {
          steps {
            script {
              echo '<--------------- Jar Publish Started --------------->'
                def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"artifactorycredentialid"
                 def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                 def uploadSpec = """{
                      "files": [
                        {
                          "pattern": "jarstaging/(*)",
                          "target": "default-maven-local/{1}",
                          "flat": "false",
                          "props" : "${properties}",
                          "exclusions": [ "*.sha1", "*.md5"]
                        }
                     ]
                 }"""
                 def buildInfo = server.upload(uploadSpec)
                 buildInfo.env.collect()
                 server.publishBuildInfo(buildInfo)
              echo '<--------------- Jar Publish Ended --------------->'
            }
          }
        }
        stage("Docker Publish") {
          steps {
            script {
               echo '<--------------- Docker Publish Started --------------->'
               docker.withRegistry(registry, 'artifactorycredentialid'){
                 docker.image(imageName).push(version)
               }
               echo '<--------------- Docker Publish Ends --------------->'
            }
          }
        }
        

    }
 }
 
