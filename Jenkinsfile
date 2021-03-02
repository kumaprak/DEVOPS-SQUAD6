
pipeline {
    agent any
    tools {
    maven 'Maven 3.6.3'
    }

    stages {
        stage('Commit Changes') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/sahadip28/DevOps-Demo-WebApp']]])
            }
        }
    
        
        // stage ('Code Quality') {
        //     steps {
        //         sh '''mvn $SONAR_MAVEN_GOAL -Dsonar.host.url=$SONAR_HOST_URL
        //         sonar.inclusions=**/test/java/servlet/createpage_junit.java
        //         sonar.test.exclusions=**/test/java/servlet/createpage_junit.java
        //         sonar.login=admin
        //         sonar.password=sonar'''
        //     }
        // }
        
        // stage ('Code Quality'){
        //     steps {
        //         withSonarQubeEnv('sonarqube') {
        //             sh 'mvn clean install -f pom.xml'
        //             sh 'mvn sonar:sonar -f pom.xml -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.sources=. -Dsonar.tests=. -Dsonar.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.test.exclusions=**/test/java/servlet/createpage_junit.java -Dsonar.login=admin -Dsonar.password=sonar'
        //         }
                
        //     }
        // }
        
        stage ('Code Quality'){
            environment {
                scannerhome = tool 'SonarQube'
            }
                steps {
                    withSonarQubeEnv('sonarqube') {

          sh "${scannerHome}/bin/sonar-scanner -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.projectKey=WEBPOC:AVNCommunication -Dsonar.sources=. -Dsonar.tests=. -Dsonar.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.test.exclusions=**/test/java/servlet/createpage_junit.java -Dsonar.login=admin -Dsonar.password=sonar"

        }


      }
        }
        
        stage('Build') {
            steps {
                sh 'mvn compile -f pom.xml'
            }
        }

        
        stage ('Deploy to Test') {
            steps{
            deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://34.122.127.79:8080/')], contextPath: '/QAWebapp', war: '**/*.war'
        }
    }
    
        stage('Store Artifacts') {
            steps{
                rtUpload (
                    serverId: 'artifactory',
                    spec: """{
                            "files": [
                                    {
                                        "pattern": "target/*.war",
                                        "target": "libs-release-local"
                                    }
                                ]
                            }"""
                )
                
                rtPublishBuildInfo (
                    serverId: 'artifactory'
                )
            }
        }
        
        stage('UI Tests') {
            steps{
                //sh "mvn test -f functionaltest/pom.xml"
                sh 'mvn test'
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test Report', reportTitles: ''])
            }
            }
            
        // stage('Performance Test'){
        //     steps{
        //       blazeMeterTest credentialsId: 'Blazemeter', testId: '9014507.taurus', workspaceId: '756616' 
        //     }
        // }
        
        stage('Deploy to Prod'){
            steps{
                deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://34.69.94.247:8080/')], contextPath: '/ProdWebapp', war: '**/*.war'
            }
        }
        
        stage('Prod-Sanity Test'){
            steps{
                //sh "mvn test -f Acceptancetest/pom.xml"
                sh 'mvn test'
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test Report', reportTitles: ''])
            }
            }
            
        stage('Slack Notification - Prod'){
            steps{
                slackSend channel: 'alerts', message: '\'Prod Build Successful\''
            }
            }
        
        
}
}
