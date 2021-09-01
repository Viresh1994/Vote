import java.text.SimpleDateFormat 
def startDate 
def endDate 
def projectId= "2" 
def projectName= "worker" 
def moduleId= "1" 
def moduleName= "Voting" 
def newBuildId= "3.0.0.${BUILD_ID}" 
def sonarUrl= "http://10.0.0.4:9000" 
def nameSpace="vote"

def CODE =  readJSON text: '{"url":"http://10.0.0.4:8888/gitlab/root/worker.git","scm":"gitlab","branch":"master","credId":"gitlab"}'
def BUILD =  readJSON text: '{"command":"docker build --tag '+moduleName+'_'+projectName+':'+newBuildId+' ."}'
def DEPLOYMENT =  readJSON text: '{"contextPath":""}'

pipeline {
agent {label 'kubernetesnode'}

	stages {

		stage('CODE'){
		        steps{
		        /*script{
					//buildName newBuildId
				}*/
		            git url: CODE.url, credentialsId: CODE.credId, branch: CODE.branch
		        }

		}
		stage('Build'){
		       steps{
			   
					println "Building image ...."
		       		script{
						   startDate = new Date()
		       		sh '''
					echo ${nameSpace}
					docker build --tag worker:${BUILD_NUMBER} .
					'''
		       		}
		       		
		       }
			   post{
				   always{
					   script{
					   		endDate = new Date()
							adoptBuildFeedback  buildDisplayName: "${BUILD_NUMBER}",
						 						buildStartedAt: "${startDate}",
						 						status: "${currentBuild.currentResult}",
						 						buildEndedAt: "${endDate}",
						 						buildUrl: "${BUILD_URL}"+"#"+CODE.url,
						 						projectId: projectId
					   }
				   }

				}
			  
		}
	   stage('Kube_deployment') {
		steps{
			script {
				sh '''
				kubectl set image deployments/worker worker=worker:${BUILD_NUMBER} --namespace=''' +nameSpace+ '''
				echo " >> Pods"
				kubectl get pods --namespace=''' +nameSpace+ '''
				'''
				}
			}
			post{
		                always{
								script{
                                		startDate = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'").format(new Date())
                                		adoptDevDeploymentFeedback  buildDisplayName: "${BUILD_NUMBER}", 
                                									deploymentStartedAt: startDate,
                                									environment: "DEV",
                                									moduleId: moduleId,
                                									moduleName: moduleName,
                                									projectName: projectName,
                                									status: "${currentBuild.currentResult}"
								}
							}
		                   
		            }
		}
	}
}

//Helper Methods

void executeCmd(String CMD){
	if(isUnix()){
		sh "echo linux"
		sh CMD
	}
	else{
		 bat "echo windows"
		 bat CMD
	}
}
