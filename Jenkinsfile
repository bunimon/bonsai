pipeline 
  {
    environment 
       {
          registryCredential = 'acrcredential'
	  apacheregistry = 'ansiblepocacr.azurecr.io/apache'
	  orderregistry = 'ansiblepocacr.azurecr.io/order'
	  catalogregistry = 'ansiblepocacr.azurecr.io/catalog'
	  customerregistry = 'ansiblepocacr.azurecr.io/customer'
          dashboardregistry = 'ansiblepocacr.azurecr.io/hystrix-dashboard'
	  apachedockerImage = ''
	  orderdockerImage = ''
	  catalogdockerImage = ''
	  customerdockerImage = ''
	  dashboarddockerImage = '' 
       }
    agent none
    stages 
       {
         stage('Build') 
	    {
	       agent 
		    { docker 
		         { image 'maven:3-jdk-8-alpine' 
		           args '-v /data/jenkins/tools/maven/.m2:/root/.m2'
                         }   
		    }
                steps 
		    {		
                        git credentialsId: 'GitHub', url: 'https://github.com/Tonyamoljose/microservice-kubernetes-demo.git'
                        sh 'mvn clean deploy sonar:sonar'      
                    }
            }
         stage('Building image') 
            {
               steps
		   {
                      script {
                              apachedockerImage = docker.build (apacheregistry + ":$BUILD_NUMBER", "./apache")
			      orderdockerImage = docker.build (orderregistry + ":$BUILD_NUMBER", "./microservice-kubernetes-demo-order")
			      catalogdockerImage = docker.build (catalogregistry + ":$BUILD_NUMBER", "./microservice-kubernetes-demo-catalog")
			      customerdockerImage = docker.build (customerregistry + ":$BUILD_NUMBER", "./microservice-kubernetes-demo-customer")
			      dashboarddockerImage = docker.build (dashboardregistry + ":$BUILD_NUMBER", "./microservice-kubernetes-demo-hystrix-dashboard")
                             }
                   }
            }
	 stage('Deploy Image to ACR') 
	    {
                steps
		    {
                      script {                     
			      docker.withRegistry( 'https://ansiblepocacr.azurecr.io', 'acrcredential' ) 
			        { 
				    apachedockerImage.push() 
				    orderdockerImage.push()
				    catalogdockerImage.push()
				    customerdockerImage.push()
				    dashboarddockerImage.push()
				}
                             }
                    }
             }
         stage('Deploy Application to AKS') 
	    {
		agent any    
	      steps
		    {
		      withCredentials([azureServicePrincipal('azurelogin')])
			  {

			    sh 'kubectl apply -f engdopdemo.yaml'
			    echo 'Waiting 2 minutes for deployment to complete'
			    sleep 120 // seconds
			    sh 'kubectl get svc'
		          }
		    }		    
            }
       }
   }
