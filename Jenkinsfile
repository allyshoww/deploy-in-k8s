pipeline {
    agent any
    parameters{
        choice(name:'Ambiente',
            choices:'srvH1\nsrvH2\nsrvH3\n',
            description: 'Qual ambiente de homologação você vai utilizar?'
            )
        choice(name:'Usuário',
            choices:'adesouzaoliv\nakofazu\n',
            description: 'Qual o usuario que você vai utilizar?'
            )
    }
    environment{
        srvH1 = '10.225.35.106'
        srvH2 = '10.225.35.170'
        user1 = 'peter-parker'
        user2 = 'tony-stark'
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t allysono/golang-helloworld${DOCKER_TAG} "
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')]) {
                    sh "docker login -u allysono -p ${dockerHubPwd}"
                    sh "docker push allysono/helloworld:${DOCKER_TAG}"
                }
            }
        }
        stage('Deploy to Kubernetes'){
            steps{
                sshagent(credentials: ['jenkins']) {
				    script{
                        if (params.Ambiente == 'srvH1'){
                            load "${params.Ambiente}/print.sh"
                            load "${params.Ambiente}/app.properties"
                            sh 'cd "srvH1/" && cp app.properties ../'
                            sh 'cat app.properties'
                            sh "ssh -o StrictHostKeyChecking=no '${user1}'@10.225.35.106 '${run.Cmd}'"
						    def runCmd = "sudo microk8s.kubectl create -f golang-helloworld.yaml"
                        } else if (params.Ambiente == 'srvH2'){
                            load "${params.Ambiente}/print.sh"
                            sh 'cd "srvH2/" && cp app.properties ../'
                            sh 'cat app.properties'
                        } else {
                            echo "Servidor errado"
                        }
					}
				}
            }
        }
    }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
