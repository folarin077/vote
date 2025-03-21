def registry= "861276124691.dkr.ecr.us-east-1.amazonaws.co"
def tag = ""
def ms = ""
def region = "us-east-1"
def repoName = "vote"
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

        stage("Deploy to Dev"){
            when{branch 'develop'}
            steps{
                script{
                    withAWS(region:"$region",credentials:'aws_creds'){
                        sh "aws eks update-kubeconfig --name vote-dev"
                        sh "kubectl set image deploy/result result=${tag} -n vote "
                        sh "kubectl rollout restart deploy/result -n vote"
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
