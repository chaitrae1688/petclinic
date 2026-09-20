pipeline {
    agent any

    tools {
        jdk 'JDK'
        maven 'maven'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage("Git Checkout") {
            steps {
                git branch: 'main',
                    changelog: false,
                    poll: false,
                    url: 'https://github.com/nimbuswiztech/Petclinic.git'
            }
        }

        stage("Compile") {
            steps {
                sh "mvn clean compile"
            }
        }

        stage("Test Cases") {
            steps {
                sh "mvn test"
            }
        }

        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Petclinic \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.projectKey=Petclinic
                    '''
                }
            }
        }

        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck(
                    additionalArguments: '--scan ./ --format HTML',
                    odcInstallation: 'DP'
                )

                dependencyCheckPublisher(
                    pattern: '**/dependency-check-report.xml'
                )
            }
        }

        stage("Build") {
            steps {
                sh "mvn clean install"
            }
        }

        stage("Publish to Artifactory") {
            steps {
                script {

                    def server = Artifactory.server('jfrog')

                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/*.war",
                                "target": "libs-release-local/org/springframework/samples/spring-framework-petclinic/5.3.13/"
                            }
                        ]
                    }"""

                    server.upload(uploadSpec)
                }
            }
        }

        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(
                        credentialsId: '58be877c-9294-410e-98ee-6a959d73b352',
                        toolName: 'docker'
                    ) {

                        sh "docker build -t image1 ."

                        sh "docker tag image1 adijaiswal/pet-clinic123:latest"

                        sh "docker push adijaiswal/pet-clinic123:latest"
                    }
                }
            }
        }

        stage("TRIVY") {
            steps {
                sh "trivy image adijaiswal/pet-clinic123:latest"
            }
        }

        stage("Deploy To Tomcat") {
            steps {
                sh "cp target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/"
            }
        }
    }
}
