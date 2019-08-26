pipeline {
 environment {
    dockerhub_repo = 'himanshuchourasia/train-schedule'
    dockerhub_credential = 'dockerhub_credentials' 
    CANARY_REPLICAS = 0
 }

    agent any
    stages {
        stage('Build') {
            steps {
            	echo "$env.BRANCH_NAME"
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
                
            }
        }
        stage('Build and push  docker image') {
           when {
               branch 'master'
               
           }
           steps{
         	script {
    			def myImage = docker.build("${dockerhub_repo}")
    			docker.withRegistry('https://registry.hub.docker.com', dockerhub_credential) {
    			myImage.push("${env.BUILD_ID}")
    			myImage.push('latest')
    		
     }
    			
}     
               
           }
       }    
           stage('Canary deployment') {
              when {
                 branch 'master'
                 
             }
             environment {
                 CANARY_REPLICAS = 1 
             }
             
             steps {
                kubernetesDeploy( 
				configs: 'train-schedule-kube-canary.yaml', 
				kubeconfigId: 'kube_config',
				enableConfigSubstitution: true
				)     			           
                 
                 
             }

           }
           stage('Smoke testing of canary') {
              steps {
                  script {
                      sleep (time: 5)
                      def response = httpRequest (
                       url: "http://$KUBE_MASTER_IP:8081",
                       timeout: 30 
                      )
                      if(response.status != 200){
                                   error("smoke test against canary has failed")          
                                         }

                  }
                  
              }

           }
           
           stage('deploy container to production'){
             when {
                 branch 'master'
                 
             }
             

             steps{
                
 				
                milestone label:'container ready for production', ordinal:1
			
				kubernetesDeploy( 
				configs: 'train-schedule-kube.yaml', 
				kubeconfigId: 'kube_config',
				enableConfigSubstitution: true
				)     			           


 				  

		
 			          
            }
         }                        
       }
       post {
           cleanup {
               	kubernetesDeploy( 
				configs: 'train-schedule-kube-canary.yaml', 
				kubeconfigId: 'kube_config',
				enableConfigSubstitution: true
				)
           }

       }
     }
   