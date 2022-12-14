// multistage
pipeline {
    agent any

        stages {
            stage('Source') {
                steps {
                    git url: 'https://github.com/Anandasb149/SpringMysqlDemo.git'
                }
            }
            stage('Build') {
                steps {
                    script {
                        def mvnHome = tool 'Maven_Home'
                        bat "${mvnHome}\\bin\\mvn -B verify"
                    }
                }
            }
             stage('Build docker image') {
                steps {
                    script {
                        bat 'docker-compose up'
                    }
                }
            }
            
            stage('SonarQube Analysis') {
                steps {
                    script {
                        def mvnHome = tool 'Maven_Home'
                        withSonarQubeEnv() {
                            bat "${mvnHome}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=SpringMysql"
                        }
                    }
                }
            }
            
       
            stage('Packaging') {
                steps {
                    step([$class: 'ArtifactArchiver', artifacts: '**/target/*.jar', fingerprint: true])
                }
            }
            
            stage ("Artifactory Publish"){
                steps{
                    script{
                            def server = Artifactory.server 'JFROG'
                            def rtMaven = Artifactory.newMavenBuild()
                            //rtMaven.resolver server: server, releaseRepo: 'jenkins-devops', snapshotRepo: 'jenkins-devops-snapshot'
                            rtMaven.deployer server: server, releaseRepo: 'Spring1', snapshotRepo: 'Spring2'
                            rtMaven.tool = 'Maven_Home'
                            
                            def buildInfo = rtMaven.run pom: '$workspace/pom.xml', goals: 'clean install'
                            rtMaven.deployer.deployArtifacts = true
                            rtMaven.deployer.deployArtifacts buildInfo

                            server.publishBuildInfo buildInfo
                    }
                }
        }
           
        }
}
