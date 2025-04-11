pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        //JAVA_HOME = "/usr/lib/jvm/java-17-openjdk-amd64"
        //PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-token', url: 'https://github.com/sef-tech/prod_cicd.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --format table -o fs.yaml .'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=prod-cicd \
                        -Dsonar.projectName=prod-cicd \
                        -Dsonar.java.binaries=target
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package'
                //sh 'mvn verify'
                //sh 'mvn install'
                //sh 'mvn clean'
                    
            }
        }

   //publish our artifact to Nexus. Add Nexus URL in the POM.xml under <distributionManagement> maven-releases and 
   //maven-snapshots.
   //Add nexus credentials in jenkins: "Manage files (installed from 'Config file provider plugin" -> "Add a new config"
   // -> 'Global Maven settings.xml' -> change the 'ID' to 'maven-settings' and click next
   // in the content box, search for  <servers>.<server>.<id>deploymentRepo</id> and create two entries of the 
   //server block, one for repo 'maven-release' and one for 'maven-snapshots' and replace the 'deploymentRepo'
   //with 'maven-release' and 'maven-snapshots' and the username and password.

        stage('Publish Artifacts to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                sh 'mvn deploy'
               }
            }
        }

        stage('Docker Build & Tag') {
            steps {
                script {
                withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                    sh 'docker build -t seftech/blogapp:latest .'
               }
               }
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --format table -o image.yaml seftech/blogapp:latest'
            }
        }
        
        stage('Docker Push Image') {
            steps {
                script {
                withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                    sh 'docker push seftech/blogapp:latest'
                  }
               }
            }
        }
        
        stage('Greeting') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
