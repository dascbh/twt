def CONTAINER_NAME="twt-app"
def CONTAINER_TAG="latest"
def DOCKER_HUB_USER="dascbh"
def HTTP_PORT="3000"

pipeline {
    agent any
    stages {
        stage('Initialize'){
            steps {
                script {
                    def scannerHome = tool 'mySonarQubeScanner';
                    def dockerHome = tool 'myDocker'
                    env.PATH = "${dockerHome}/bin:${scannerHome}/bin:${env.PATH}"
                }
            }
        }
        stage('Checkout'){
            steps {
                checkout scm
            }
        }
        stage('Build'){
            agent {
                docker { 
                    image 'node'
                    args '-v /var/jenkins_home/workspace/twt-app-hapi-nodejs:/srv/twt-app-hapi-nodejs' 
                }
            }
            steps {
                script {
                    sh 'cd src && npm install'
                }
            }
        }
        stage('Unit Tests'){
            agent {
                docker { 
                    image 'node'
                    args '-v /var/jenkins_home/workspace/twt-app-hapi-nodejs:/srv/twt-app-hapi-nodejs' 
                }
            }
            steps {
                script {
                    sh 'cd src && npm run test'
                }
            }
        }
        stage('SonarQube Analysis'){
            steps {
                withSonarQubeEnv('mySonarQubeServer') {
                    sh "sonar-scanner"
                }
            }
        }
        stage("Image Prune"){
            steps {
                imagePrune(CONTAINER_NAME)
            }
        }
        stage('Image Build'){
            steps {
                imageBuild(CONTAINER_NAME, CONTAINER_TAG)
            }
        }
        stage('Run App'){
            steps {
                runApp(CONTAINER_NAME, CONTAINER_TAG, DOCKER_HUB_USER, HTTP_PORT)
            }
        }
    }
}

def imagePrune(containerName){
    try {
        sh "docker image prune -f"
        sh "docker stop $containerName"
    } catch(error){}
}

def imageBuild(containerName, tag){
    sh "docker build -t $containerName:$tag  -t $containerName --pull --no-cache ./demo"
    echo "Image build complete"
}

def pushToRegistry(containerName, tag, dockerUser, dockerPassword){
    sh "docker login -u $dockerUser -p $dockerPassword"
    sh "docker tag $containerName:$tag $dockerUser/$containerName:$tag"
    sh "docker push $dockerUser/$containerName:$tag"
    echo "Image push complete"
}

def runApp(containerName, tag, dockerHubUser, httpPort){
    sh "docker pull $dockerHubUser/$containerName"
    sh "docker run -d --rm -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag"
    echo "Application started on port: ${httpPort} (http)"
}