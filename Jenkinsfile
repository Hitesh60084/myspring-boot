pipeline{
    agent any
    environment { 
                  registry1 = "519852036875.dkr.ecr.us-east-2.amazonaws.com/cloudjournee:${env.BUILD_NUMBER}"
                }
    tools {maven "MAVEN"}
    stages{
        stage('code checkout from GitHub'){
            steps{
                //check out code from the GitHub
                git branch: 'main', url: 'https://github.com/Abhilash-1201/myspring-boot-dev.git'
            }
        }
        //This stage gets all code Quality check from the GitHub Repository
        stage('Code Quality Check via SonarQube'){
            steps{
                script{
                    def scannerHome = tool 'sonarqube-scanner';
                    withSonarQubeEnv(credentialsId: 'sonarqube_access_token'){
                        if(fileExists("sonar-project.properties")) {
                         sh "${tool("sonarqube-scanner")}/bin/sonar-scanner"
                             
                         }  
                        
                    }
                }
            }
        }
        stage('Build') {
            steps {
                // Build the Maven code after analysis
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }
        // Build the docker image to store in to ECR
        stage('Building docker image for dev')  {
         steps{
           script{
               dockerImage = docker.build registry1
           }
         }
       }
        // Push the docker image in to dev ECR
       stage('Pushing docker image to Dev-ECR') {
        steps{  
         script {
                sh 'docker logout'
                sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 519852036875.dkr.ecr.us-east-2.amazonaws.com'
             sh 'docker push ${registry1}'
               }
           }
      
        }  
        //Deploy docker image in to dev eks 
       stage ('K8S Deploy') {
       steps { 
                kubernetesDeploy(
                    configs: 'springboot-lb.yaml',
                    kubeconfigId: 'k8s',
                    enableConfigSubstitution: true
                    )               
             }  
         }
        //Conformation to production after approval
        stage('Prod Approval confirmation') {
            steps{
            script {
                        env.RELEASE_TO_PROD = input message: 'Do you want to create prod build?',
                            parameters: [choice(name: 'Promote to production', choices: 'No\nYes', description: 'Choose "yes" if you want to deploy this build in production')]
                        milestone 1
                    }
            }

        }
         // Build the docker image to store in to Prod ECR
        stage('Building docker image for prod')  {
         steps{
           script{
               dockerImage = docker.build registry1
           }
         }
       }
         // Push the docker image in to prod ECR
       stage('Pushing docker image to Prod-ECR') {
        steps{  
         script {
                //sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 519852036875.dkr.ecr.us-east-2.amazonaws.com'
                sh 'docker push ${registry1}'
               }
           }
      
        }  
//         //Deploy docker image in to prod eks 
//        stage ('K8S Deploy') {
//        steps { 
//                 kubernetesDeploy(
//                     configs: 'springboot-lb.yaml',
//                     kubeconfigId: 'k8s',
//                     enableConfigSubstitution: true
//                     )               
//              }  
//          }
        //Email notification after build get successful
        stage('Build success email notification ') {
          steps {
            mail to: "digin@cloudjournee.com",
                 cc: "abhilash.rl@cloudjournee.com",
                subject: "SUCCESSFUL: Build ${env.JOB_NAME} ${env.BUILD_NUMBER}",
                body: "Build Successful!! Build ${env.JOB_NAME} with ${env.BUILD_NUMBER}\n\n\nBuild Name:  ${env.JOB_NAME}\nBuild Number: ${env.BUILD_NUMBER}\nJenkins URL: ${env.BUILD_URL}\n\nClick below link to proceed to prod environment\n\nhttp://ec2-18-119-103-227.us-east-2.compute.amazonaws.com:8080/job/Pipeline%20-%20prod%204/buildWithParameters?token=123456&BuildNumber=${env.BUILD_NUMBER}"
            }
        }     
    }
   
}
