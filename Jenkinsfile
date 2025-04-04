def registry= "861276124691.dkr.ecr.us-east-1.amazonaws.com"
def tag = ""
def ms = ""
def region = "us-east-1"

pipeline{
    agent any
    stages{
        stage("init"){
            steps{
                script{
                    tag = getTag()
                    ms = getMsName()
                }
            }
        }
        stage("Build Docker image"){
            steps{
                script{
                    sh "docker build . -t ${registry}/${ms}:${tag}"
                }
            }
        }

        stage("Login to Ecr"){
            steps{
                script{
                    withAWS(region:"$region",credentials:'aws_creds'){
                        sh "aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${registry}"
                    }
                }
            }
        }

        stage("Docker push"){
            steps{
                script{
                    withAWS(region:"$region",credentials:'aws_creds'){
                        sh "docker push ${registry}/${ms}:${tag}"
                    }
                }
            }
        }

        stage("Create EKS Cluster") {
            steps{
                script{
                    withAWS(region: "$region", credentials: 'aws_creds') {
                // Check if the cluster already exists (optional, avoids re-creating)
                        def clusterExists = sh(script: "eksctl get cluster --name vote-dev --region ${region} || echo 'Cluster does not exist'", returnStdout: true).trim()
                
                        if (clusterExists.contains('Cluster does not exist')) {
                    // Create the EKS cluster if it doesn't exist
                            sh """
                            eksctl create cluster --name vote-dev --region ${region} --nodegroup-name my-nodes --node-type t3.small --managed --nodes 2
                            """
                            echo "EKS Cluster 'vote-dev' created successfully."
                        } else {
                            echo "EKS Cluster 'vote-dev' already exists."
                        }
                    }
                }
            }
        }

        
        stage("Deploy to Dev"){
            when{branch 'develop'}
            steps{
                script{
                    withAWS(region:"$region",credentials:'aws_creds'){
                        sh "aws eks update-kubeconfig --name vote-dev"
                        sh 'curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.28.5/2024-01-04/bin/linux/amd64/kubectl'  
                        sh 'chmod u+x ./kubectl'
                        sh "kubectl get deployment vote -n vote || kubectl apply -f k8s/deployment.yaml -n vote"
                        sh "kubectl set image deploy/vote vote=${registry}/${ms}:${tag} -n vote "
                        sh "kubectl rollout restart deploy/vote -n vote"
                    }
                }
            }
        }
    }
}

def getMsName(){
    print env.JOB_NAME
    return env.JOB_NAME.split("/")[0]
}

def getTag(){
 sh "ls -l"
 version = "1.0.0"
 print "version: ${version}"

 def tag = ""
  if (env.BRANCH_NAME == "main"){
    tag = version
  } else if(env.BRANCH_NAME == "develop"){
    tag = "${version}-develop"
  } else {
    tag = "${version}-${env.BRANCH_NAME}"
  }
return tag 
}
