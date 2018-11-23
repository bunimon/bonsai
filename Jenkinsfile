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
                              apachedockerImage = docker.build (apacheregistry + ":$BUILD_NUMBER", "apache/Dockerfile")
			      orderdockerImage = docker.build (orderregistry + ":$BUILD_NUMBER", "./microservice-kubernetes-demo-order/Dockerfile")
			      catalogdockerImage = docker.build (catalogregistry + ":$BUILD_NUMBER", ".microservice-kubernetes-demo-catalog/Dockerfile")
			      customerdockerImage = docker.build (customerregistry + ":$BUILD_NUMBER", "./microservice-kubernetes-demo-customer/Dockerfile")
			      dashboarddockerImage = docker.build (dashboardregistry + ":$BUILD_NUMBER", "./microservice-kubernetes-demo-hystrix-dashboard/Dockerfile")
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
				    hystrix-dashboarddockerImage.push()
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
			    sh 'kubectl get svc'
		          }
		    }		    
            }
       }
   }
