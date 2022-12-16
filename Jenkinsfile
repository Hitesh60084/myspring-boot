pipeline{
  agent {label 'login_page'}
    stages{
        stage('CHECKOUT GIT'){
            steps{
                git branch: 'main', url: 'https://github.com/nayab786910/myspring-boot.git'
            }
        }
        stage('CODE QUALITY'){
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
    }
}
