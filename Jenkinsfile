def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {
    agent any
    
    
    environment{
        WORKSPACE = "${env.WORKSPACE}"
    }
    
    
    tools {
        maven 'localMaven'
        jdk 'localJdk'
    }
    
    
    stages {
        stage('Git Checkout') {
            steps {
                echo 'Cloning the application code...'
                git branch: 'main', url: 'https://github.com/Dreo57/Jenkins-CICD-pipeline-project-win.git'
            }
        }
        stage('Build') {
            steps {
                sh 'java -version'
                sh 'mvn clean package'
            }
            post {
                success {
                    echo'archiving...'
                    archiveArtifacts artifacts: '**/*.war', followSymlinks: false
                }
            }
        }
        stage('Unit Test'){
            steps {
                sh 'mvn test'
            }
        }
        stage('Integration Test'){
            steps {
              sh 'mvn verify -DskipUnitTests'
            }
        }
        stage ('Checkstyle Code Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
        stage('SonarQube Scan') {
          steps {
              withSonarQubeEnv('SonarQube') {
                  
                    sh """
                        mvn clean verify sonar:sonar \
                          -Dsonar.projectKey=JavaWebApp \
                          -Dsonar.host.url=http://localhost:9000 \
                          -Dsonar.login=sqp_ac5790170a648157ce97cdc715bb0ad588b4fdb8                      
                    """
              }
           }
        }
        stage("Quality Gate"){
             steps{
                 waitForQualityGate abortPipeline: true
                 
             }
             
        }
        stage('Upload artifact to nexus') {
            steps {
                sh 'mvn clean deploy -DskipTests'
            }
        }        
        stage('Deploy to DEV') {
          environment {
            HOSTS = "dev"
          }
          steps {
            sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
          }
         }        
        stage('Deploy to STAGE env') {
          environment {
            HOSTS = "stage"
          }
          steps {
            sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
          }
         }         
        stage('Approval') {
          steps {
            input("Do you want to proceed?")
          }
        }         
        stage('Deploy to Prod env') {
          environment {
            HOSTS = "prod"
          }
          steps {
            sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
          }
         }
    }
    post { 
        always { 
            echo 'I will always say Hello again!'
                slackSend channel: '#glorious-w-f-devops-alerts', color: COLOR_MAP[currentBuild.currentResult], message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}
