def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {
    
    agent {
        label 'Slave1'
    }

    tools 
    {
        maven 'Maven-3.8.7'
    }
    
    stages {
        stage('SCM-Checkout GIT') {
            steps {
                git 'https://github.com/kunalchinchole/star-agile-health-care.git'
            }
		}
			
        stage('MAVEN Build') {
            steps {
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
		}
          
        stage('Build DockerFile') {
            steps {
                sh 'docker rmi kunalchinchole121/healthcare'
                sh "docker build -t kunalchinchole121/healthcare:${BUILD_NUMBER} ."
                sh "docker tag kunalchinchole121/healthcare:${BUILD_NUMBER} kunalchinchole121/healthcare:latest"
		//latest
                sh 'docker image list'
                sh "docker rmi kunalchinchole121/healthcare:${BUILD_NUMBER}"
                sh 'docker image list'
            }
        }
        
        stage('Docker_HUB_Login & Push Image') {
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'dockerhubuidpid', passwordVariable: 'dockerhubpwd', usernameVariable: 'dockerhubun')]) {
                        sh 'docker login -u ${dockerhubun} -p ${dockerhubpwd}'
                    }
                        sh 'docker push kunalchinchole121/healthcare:latest'
                }
            }
        }

        stage('Deploy to Kubernetes Cluster') {
            steps {
		        script {
                    sshPublisher(publishers: [sshPublisherDesc(configName: 'Ks9-Master', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '.', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'healthcare-deploy.yaml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
		        }
            }
	    }

//         stage('Approve - kr9 Diployment') {
//             steps{              
//                 //----------------send an approval prompt-------------
//                 script {
//                    env.APPROVED_DEPLOY = input message: 'User input required Choose "yes" | "Abort"'
//                        }
//                 //-----------------end approval prompt------------
//             }
//         }

        stage('Kubernetes app Deploy') {
            agent {
                label 'Slave2'
            }
            steps {
                    sh 'mv /root/healthcare-deploy.yaml /root/workspace/Healthcare_Domain'
                    sh 'kubectl get po -o wide'
                    sh 'kubectl delete -f healthcare-deploy.yaml'
                    sh 'kubectl get po -o wide'
                    sh 'kubectl create -f healthcare-deploy.yaml'
                    sh 'kubectl get po -o wide'
                    //sh 'kubectl apply -f healthcare-deploy.yaml'
                    //sh 'kubectl rollout status deployment/healthcare-deploy'
                    //sh 'kubectl get po -o wide'
                    sh 'kubectl get svc'
                }
        }
	}

    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}
