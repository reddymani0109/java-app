pipeline {
    agent any
    tools {
        maven "maven"
    }
    stages {
        stage("Checkout") {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'github-creds', url: 'https://github.com/reddymani0109/java-app.git']])
            }
        }
        stage("Build") {
            steps {
                sh '''
                    mvn clean install -f MyWebApp/pom.xml
                '''
            }
        }
        stage("Code Quality") {
            steps {
                withSonarQubeEnv(installationName: 'sonar', credentialsId: 'sonar-jenkins-token') {
                    sh '''   
                     mvn sonar:sonar -f MyWebApp/pom.xml;
                   
                   ''';
                }
            }
        }
        stage("Nexus Upload") {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '65.0.168.147:9081/',
                    groupId: 'com.mkyong',
                    version: '1.0-SNAPSHOT',
                    repository: 'maven-snapshots',
                    credentialsId: 'nexus-creds',
                    artifacts: [
                        [
                            artifactId: 'MyWebApp',
                            classifier: '',
                            file: 'MyWebApp/target/MyWebApp.war',
                            type: 'war'
                        ]
                    ]
                )
            }
        }
        stage("Dev Deploy") {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-creds', path: '', url: 'http://13.233.227.170:8080/')], contextPath: null, war: '**/*.war'
            }
        }
        stage("QA Approval") {
            steps {
                echo "Waiting for the approval for QA deployement.."
                timeout(time: 7, unit: 'HOURS') {
                    input (
                        message: "Do you want to deploy to QA?",
                        ok: "Yes,Go Ahead!",
                        submitter: "jenkins,admin"
                    )
                }
            }
        }
        stage("QA Deploy") {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-creds', path: '', url: 'http://13.233.227.170:8080/')], contextPath: null, war: '**/*.war'
            }
        }
    }
}


