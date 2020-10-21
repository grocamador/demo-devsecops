pipeline {
    agent any
    
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "grocamador/train-schedule"
        CHKP_CLOUDGUARD_ID = credentials("chkp-id")
       CHKP_CLOUDGUARD_SECRET = credentials("chkp-key")
       SG_CLIENT_ID = credentials("sg-client")
        SG_SECRET_KEY = credentials("sg-secret")
        KUBECONFIG = credentials("my-kubeconfig")

        }
    
    stages {
        
       stage('ShiftLeft Code Scan') {   
            steps {   
            echo 'Skipping codescan'    
 //                  script {      
 //                       try {
 //                           sh 'chmod +x shiftleft' 
 //                           sh './shiftleft code-scan -s .'
 //                       } catch (Exception e) {
 //                           echo "Request for Approval"  
 //                       }
 //                  }
            }
         
         }
        
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy to stage') {
            when {
                branch 'master'
            }
            steps {
            try {
             sh 'kubectl delete -f train-schedule-kube-stage.yml'
            } catch (Exception e) {
             sh 'echo "Not existing yet in stage"'
            }
            sh ("""     
                  kubectl apply -f train-schedule-kube-stage.yml
                """)
 
                // Other tentative that didnt work:
 //            kubectl set image deployment/train-schedule-deployment-stage train-schedule-stage=grocamador/train-schedule
 //            kubectl set image deployment/train-schedule-deployment-stage train-schedule-stage=grocamador/train-schedule:latest
            }
        }
        
       stage("Deploy to Production"){
            when {
                branch 'master'
            }
             steps {
                 
                input 'Deploy to Production?'
                  milestone(1)
//              With KUBECTL and Kubeconfig       
             try {
             sh 'kubectl delete -f train-schedule-kube.yml'
            } catch (Exception e) {
             sh 'echo "Not existing yet in prod"'
            }
              sh ("""                
                  echo \$KUBECONFIG
                  kubectl apply -f train-schedule-kube.yml
                """)

                 
//                 kubernetesDeploy(
//                    kubeconfigId: 'kubeconfig',
//                    configs: 'deploy.yml',
//                    enableConfigSubstitution: true
//                )

                
             }
         }
    }
}
