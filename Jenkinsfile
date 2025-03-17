def registry= "557690600815.dkr.ecr.eu-west-2.amazonaws.com"
def tag = ""
def ms = ""
def region = "eu-west-2"

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

  def branchName = env.BRANCH_NAME ?: "unknown"
    def tag = ""

    if (branchName == "main"){
        tag = version
    } else if(branchName == "develop"){
        tag = "${version}-develop"
    } else if (branchName != "unknown") {
        tag = "${version}-${branchName}"
    } else {
        error("BRANCH_NAME is not set properly!")
    }

    print "Final tag: ${tag}"
return tag 
}
