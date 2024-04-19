pipeline {
    agent any
    
    tools {
        maven "M3"
        dockerTool 'docker'
    }

    environment {
        imageName = "skistation:0.0.1"
        registryUsername= "admin"
        registryCredentials = "Nexus"
        registry = "localhost:8085"
        dockerImage = ''
    }

    stages {
        stage('Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/jglick/simple-maven-project-with-tests.git'
            }
        }

        // stage('Build') {    
        //     steps {
        //         // Run Maven on a Unix agent.
        //         sh "mvn -Dmaven.test.failure.ignore=true clean package"

        //         // To run Maven on a Windows agent, use
        //         // bat "mvn -Dmaven.test.failure.ignore=true clean package"
        //     }

        //     post {
        //         // If Maven was able to run the tests, even if some of the test
        //         // failed, record the test results and archive the jar file.
        //         success {
        //             junit '**/target/surefire-reports/TEST-*.xml'
        //             archiveArtifacts 'target/*.jar'
        //         }
        //     }
        // }
        
        stage('sonarqube'){
            agent { 
                docker { 
                    image 'sonarsource/sonar-scanner-cli' 
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                } 
            }
            steps{
                sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=skistation\
                    -Dsonar.sources=. \
                    -Dsonar.host.url=http://172.19.0.4:9000 \
                    -Dsonar.login=admin \
                    -Dsonar.password=000000
                '''
                }
            
        }

        stage('dockerize the app') {
            steps {
                sh 'echo dockerize'
                sh 'docker build -t ${{registry}}/${{imageName}}'
            }
        }

        stage('Uploading to Nexus') {
            steps {
                sh 'docker login ${{registry}} -u ${{registryCredentials}} -p ${{registryUsername}}'
                sh 'docker push ${{registry}}/${{imageName}}'
            }
        }
    }
}