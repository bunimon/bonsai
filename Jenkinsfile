pipeline 
  {
    environment 
       {
	  resourcegroup = 'RG_MSDP'
	  aksname = 'MSDPaks'
	  acrservername = 'msdpacr.azurecr.io'
          registryCredential = 'acrcredential'
	  apacheregistry = 'msdpacr.azurecr.io/apache'
	  orderregistry = 'msdpacr.azurecr.io/order'
	  catalogregistry = 'msdpacr.azurecr.io/catalog'
	  customerregistry = 'msdpacr.azurecr.io/customer'
          dashboardregistry = 'msdpacr.azurecr.io/hystrix-dashboard'
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
			      docker.withRegistry( 'https://msdpacr.azurecr.io', 'acrcredential' ) 
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
			    sh 'sed -i s,version,${BUILD_NUMBER},g deployment.yml'
                            sh 'az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID'
			    sh 'az aks get-credentials --resource-group $resourcegroup --name $aksname'
			    sh 'kubectl apply -f deployment.yml'
			    echo 'Waiting for external IP to be genarated'
			    sleep 120 // seconds
			    sh 'kubectl get svc'
		          }
		    }		    
            }
       }
   }
