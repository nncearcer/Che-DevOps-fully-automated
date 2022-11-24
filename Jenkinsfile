def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]



pipeline {
    agent any
    
    environment{
       
        WORKSPACE = "${env.WORKSPACE}"
        
    }
    
    tools{
        maven 'localMaven'
        jdk 'localJdk'
    }
    

    stages {
        stage('Git checkout') {
            steps {
                echo 'Cloning the application code...'
                git branch: 'main', url: 'https://github.com/cvamsikrishna11/devops-fully-automated.git'                
                sh 'ls'
                
                
            }
        }
        
        
        stage('Build') {
            steps {
                sh 'java -version'
                sh 'mvn clean package'
                
            }
        
            post {
                success {
                    echo 'archiving....'
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
    
    
    stage ('SonarQube scanning'){
        steps {
            
            withSonarQubeEnv('SonarQube') {
            
            sh """
          mvn sonar:sonar \
            -Dsonar.projectKey=maven \
            -Dsonar.host.url=http://172.31.14.9:9000 \
            -Dsonar.login=9bb799d032fda12f2234c7686dd30b8459659aa9
            
            """
            }
        }
        
    }
    
    
  stage("Quality Gate"){
    
      steps{
           
      waitForQualityGate abortPipeline: true
           
         }
        
    }
    
    
    
    stage("Upload artifact to Nexus"){
    
      steps{
            sh 'cp settings.xml /var/lib/jenkins/.m2'
            sh 'mvn clean deploy -DskipTests'
           
         }
        
    }
    
    
    
    stage('Deploy to DEV') {
      environment {
        HOSTS = "dev"
      }
      steps {
        sh 'ls'
        sh 'cp ansible/aws_ec2.yml /opt/ansible/inventory/'
        // sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
        sh "ansible-playbook -i /opt/ansible/inventory/aws_ec2.yaml ${WORKSPACE}/deploy.yaml --extra-vars \"ansible_user=ansadmin ansible_password=ansadmin hosts=tag_Environment_dev workspace_path=$WORKSPACE\""
      }
     }
     
     
    stage('Deploy to STAGE env') {
      environment {
        HOSTS = "stage"
      }
      steps {
        sh 'ls'
        // sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
        sh "ansible-playbook -i /opt/ansible/inventory/aws_ec2.yaml ${WORKSPACE}/deploy.yaml --extra-vars \"ansible_user=ansadmin ansible_password=ansadmin hosts=tag_Environment_stage workspace_path=$WORKSPACE\""
        
      }
     }
     
     
    
     stage('Approval') {
      steps {
        input('Do you want to proceed?')
      }
    } 
     
     
     
    stage('Deploy to PROD env') {
      environment {
        HOSTS = "prod"
      }
      steps {
        sh 'ls'
        // sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
        sh "ansible-playbook -i /opt/ansible/inventory/aws_ec2.yaml ${WORKSPACE}/deploy.yaml --extra-vars \"ansible_user=ansadmin ansible_password=ansadmin hosts=tag_Environment_prod workspace_path=$WORKSPACE\""
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