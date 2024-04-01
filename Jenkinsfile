pipeline {
    agent any

    tools {
        maven "maven3.9.6"
    }

    stages {
        stage("git clone") {
            steps {
                git 'https://github.com/SirDominique/EcommerceApp.git'
            }
        }

        stage("build,Test and Package with maven") {
            steps {
                // Specify the directory containing the pom.xml file
                dir('EcommerceApp') {
                    sh '''
                    mvn clean
                    mvn test
                    mvn package
                    '''
                }
                
            }
        }

        stage ('SonarQube Analysis') {
            environment {
                ScannerHome = tool "sonar5.0"
            }

            steps {
                echo "Analyzing with SonarQube.."
                script {
                    dir ('EcommerceApp') {
                        compiledClassesDir = sh(script: 'mvn help:evaluate -Dexpression=project.build.outputDirectory -q -DforceStdout', returnStdout: true).trim()

                        withSonarQubeEnv('sonarqube') {
                            sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=ecommercefrontend -Dsonar.java.binaries=${compiledClassesDir}"
                        }

                    }
                }
            }

        }

        stage ('Upload to Nexus') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'EcommerceApp', classifier: '', file: '/var/lib/jenkins/workspace/Ecommerce-app/EcommerceApp/target/EcommerceApp.war', type: 'war']], credentialsId: 'nexus-id', groupId: 'com', nexusUrl: '35.94.144.94:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'Ecommerceapp-snapshot', version: '0.0.1-SNAPSHOT'
            }
        }

        stage ('Deploy to Tomcat') {
            steps {
                dir ('EcommerceApp') {
                   deploy adapters: [tomcat9(credentialsId: 'tomcat-credentials', path: '', url: 'http://34.219.120.95:8080/')], contextPath: null, war: 'target/*.war'
                }

            }
        }
            
        
    }
}