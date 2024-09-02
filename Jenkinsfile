pipeline {
  agent any
  tools { 
        maven 'Maven_3_2_5'  
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=devsecopsbuggywebapplication -Dsonar.organization=devsecopsbuggywebapplication -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=404c00e0d0f7def97b915dbba0fd0e3435870bb2'
			}
    }

	stage('RunSCAAnalysisUsingSnyk') {
            steps {		
				withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
    }

	stage('Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 app =  docker.build("devsecopsimage") // Create a repo on AWS ECR that must match the imagename
                 }
               }
            }
    }

	stage('Push') {
            steps {
                script{
                    docker.withRegistry('https://559050205784.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-west-2:aws-credentials') {
                    app.push("latest")
                    }
                }
            }
    	}
	   
	stage('Kubernetes Deployment of ASG Bugg Web Application') {
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) {
		  sh('kubectl delete all --all -n devsecops')
		  sh ('kubectl apply -f deployment.yaml --namespace=devsecops') // Create the namespace beforehand using "kubectl create namespace devsecops"
		}
	      }
   	}

  }
}
// to verify deployment run the following command on EC2 connect : kubectl get deployments -n devsecops
// to verify loadbalancer configuration run the following command : kubectl get svc -n devsecops
