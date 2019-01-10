pipeline{
    agent any
    
    parameters {
         string(name: 'tomcat_stg', defaultValue: '18.191.249.244', description: 'Staging Server')
         string(name: 'tomcat_prod', defaultValue: '3.17.153.45', description: 'Production Server')
    }
    
    triggers {
         pollSCM('* * * * *')
     }
    
    stages{
        stage('Build'){
            steps{
                sh 'mvn clean package'
            }
            post{
                success{
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }
        stage ('Deploy to Staging'){
            steps {
                sh "ssh-keyscan -H ${params.tomcat_stg} >> ~/.ssh/known_hosts"
                sh "scp -i /tomcat-demo.pem **/target/*.war ec2-user@${params.tomcat_stg}:/var/lib/tomcat7/webapps"
            }
        }
        stage ('Deploy to Production'){
            steps {
                timeout(time:5, unit: 'DAYS'){
                    input message: 'Approve PROD deployment?'
                }
                
                sh "ssh-keyscan -H ${params.tomcat_prd} >> ~/.ssh/known_hosts"
                sh "scp -i /tomcat-demo.pem **/target/*.war ec2-user@${params.tomcat_prd}:/var/lib/tomcat7/webapps"
            }
            post {
                success{
                    echo 'Code successfully deployed to Production'
                }

                failure{
                    echo 'Deployment failed'
                }
            }
        }
    }
}
