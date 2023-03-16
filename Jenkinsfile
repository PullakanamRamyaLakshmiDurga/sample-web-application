currentBuild.displayName = "Final_Demo # "+currentBuild.number

   def getDockerTag(){
        def tag = sh script: 'git rev-parse HEAD', returnStdout: true
        return tag
        }
        

pipeline{
        agent any  
        environment{
	    Docker_tag = getDockerTag()
        }
        
        stages{


              stage('Quality Gate Statuc Check'){

               agent {
                docker {
                image 'maven'
                args '-v $HOME/.m2:/root/.m2'
                }
            }
                  steps{
                      script{
                      withSonarQubeEnv(installationName: 'sonar-durga') { 
                      sh "mvn sonar:sonar"
                       }
                      timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
		    sh "mvn clean install"
                  }
                }  
              }



              stage('build')
                {
              steps{
                  script{
		 sh 'cp -r ../durga88@2/target .'
                   sh 'docker build . -t durgapullakanam/docker:$Docker_tag'
		   withCredentials([string(credentialsId: 'docker_password', variable: 'docker_password')]) {
				    
				  sh 'docker login -u durgapullakanam -p $docker_password'
				  sh 'docker push durgapullakanam/docker:$Docker_tag'
			}
                       }
                    }
                 }
		 
		stage('ansible playbook'){
			steps{
			 	script{
				    sh '''final_tag=$(echo $Docker_tag | tr -d ' ')
				     echo ${final_tag}test
				     sed -i "s/docker_tag/$final_tag/g"  deployment.yaml
				     '''
				     ansiblePlaybook credentialsId: 'fa4f0834-9765-4edb-8f78-8998290f383f', installation: 'ansible', inventory: 'hosts', playbook: 'ansible.yaml'
				    
				     //ansiblePlaybook installation: 'ansible', inventory: 'hosts', playbook: 'ansible.yaml'
					//sh 'ansible-playbook -i ./hosts/ ansible.yaml'
				}
			}
		}
		
	
		
               }
	       
	       
	       
	      
    
}
