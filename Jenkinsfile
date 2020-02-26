def registry = "allysono/deploy-in-k8s"
def registryCredential = 'docker-hub'

def command = """ls -lah"""
def proc = command.execute()
proc.waitFor()
def list=proc.in.text.readLines() 

def listServers = [
 srvH1: [name:"srvH1", user: 'peter-parker', ip: "10.225.35.106"],
 srvH2: [name:"srvH2", user: 'tony-stark', ip: "10.225.35.107"],
 srvH3: [name:"srvH3", user: 'hulk', ip: "10.225.35.108"]
]  

pipeline {
    agent any
    
    parameters{
        choice(name:'Ambiente',
            choices:'srvH1\nsrvH2\nsrvH3\n',
            description: 'Qual ambiente de homologação você vai utilizar?'
            )
        choice(name:'User',
            choices:'peter-parker\ntony-stark\nhulk\n',
            description: 'Qual usuário você vai utilizar?'
            )
        choice(name: "teste",
            choices:${list},
            description: 'Teste')
    }
    environment{
        srvH1 = "${listServers.srvH1.name}"
        srvH2 = "${listServers.srvH2.name}"
        srvH3 = "${listServers.srvH3.name}"
 
    }

 stages{
        stage ('Clone do repositório') {
            steps{
            sh 'git clone https://github.com/allysono/deploy-in-k8s.git && cd deploy-in-k8s'
            }
        }

        stage('Docker Build  Image'){
            steps{
                script {
                    appimage = docker.build registry + ":" + getDockerTag()
                }
            }
        }
        stage('Docker Push  Image'){
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                        appimage.push()
                        appimage.push('latest')
                   }
               }
           }
        }
        
        stage('Deploy to Kubernetes'){
            steps{
                sshagent(credentials: ['jenkins']) {
				    script{
                            sh "cp deploy/${params.Ambiente}/*.properties app.properties"
                            sh "ssh -o StrictHostKeyChecking=no ${listServers.find{ it.key == params.Ambiente }?.value.user}@${listServers.find{ it.key == params.Ambiente }?.value.ip} -p 22 'kubectl create -f golang-helloworld.yaml'"
                            sh 'cat app.properties'
                       
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