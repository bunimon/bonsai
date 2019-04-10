pipeline 
  {
    agent none
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
			agent any
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
				agent any
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
         stage('Deploy Application to DEV') 
	    {
		agent any    
	      steps
		    {
		      withCredentials([azureServicePrincipal('azurelogin')])
			  {
			    sh 'sed -i s,version,${BUILD_NUMBER},g deployment.yml'
                            sh 'az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID'
			    sh 'az aks get-credentials --resource-group $resourcegroup --name $aksname'
				sh 'kubectl create namespace dev-micro'
			    sh 'kubectl apply -f deployment.yml --namespace=dev-micro'
			    echo 'Waiting for external IP to be genarated'
			    sleep 120 // seconds
			    sh 'kubectl get svc --namespace=dev-micro'
		          }
		    }		    
            }
			              stage ('QA Promotion')
              {

                  steps
                  {
                      
                        emailext (
            subject: "QA Deployment Approval for micro app",
            body: """Hi Team,
            
Dev verification for the micro_app has been completed. 
            
Could you please Approve/Reject the micro_app to QA usning the below link
            
${env.BUILD_URL}input 
           
Regards,
Jenkins""",
            to: 'aromalraj123@gmail.com'
            
          )
        
                input message: "Do you want to approve this job to deploy to QA?", submitter: "jenkins"
                

                    }
              }
		 stage('Deploy Application to QA') 
	    {
		agent any    
	      steps
		    {
		      withCredentials([azureServicePrincipal('azurelogin')])
			  {
			    sh 'sed -i s,version,${BUILD_NUMBER},g deployment.yml'
                            sh 'az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID'
			    sh 'az aks get-credentials --resource-group $resourcegroup --name $aksname'
				sh 'kubectl create namespace qa-micro'
			    sh 'kubectl apply -f deployment.yml --namespace=qa-micro'
			    echo 'Waiting for external IP to be genarated'
			    sleep 120 // seconds
			    sh 'kubectl get svc --namespace=qa-micro'
		          }
		    }		    
            }

       }
   }
