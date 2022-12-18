pipeline{
  agent {label 'login_page'}
      environment {
        registry = "519852036875.dkr.ecr.us-east-2.amazonaws.com/cloudjournee:latest"
    }
    stages{
        stage('CHECKOUT GIT'){
            steps{
                git branch: 'main', url: 'https://github.com/nayab786910/myspring-boot.git'
            }
        }
        stage('CODE QUALITY'){
            steps{
                script{
                    withSonarQubeEnv('sonarqube_cred'){
                        if(fileExists("sonar-project.properties")) {
                         sh "mvn sonar:sonar"
                         }
                    }
                }
            }
        }
        stage('BUILDING ARTIFACT'){
          agent { label 'login_page'}
     			 steps{
        			  echo 'build '
                sh "mvn clean package"
     			  }
        }
        stage('DOCKER IMAGE') 
       {
          agent { label 'login_page'}
          steps{
              script {
                myImage = docker.build registry
              }
           }
        }
      stage('Pushing to ECR') {
         agent { label 'login_page'}
         steps{  
         script {
                sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 519852036875.dkr.ecr.us-east-2.amazonaws.com'
                sh 'docker push 519852036875.dkr.ecr.us-east-2.amazonaws.com/cloudjournee:latest'
                }
          }
      }
      stage ('K8S Deploy') {
          steps { 
                kubernetesDeploy(
                    configs: 'springboot.yaml',
                    kubeconfigId: 'k8s',
                    enableConfigSubstitution: true
                    )               
                }  
       }
  }
}
