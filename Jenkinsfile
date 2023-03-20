pipeline {

agent any

stages {

stage('Git checkout') {

steps {

 git 'https://github.com/gitcodejo/java.git'

}

}

stage('unit testing') {

steps {

 sh 'mvn test'

}

}


stage('integration testing') {

steps {

 sh 'mvn verify -DskipUnitTests'

}

}

stage('maavan build') {

steps {

sh 'mvn clean install'

}

}

stage('static code analysis') {

steps {
    script{
      withSonarQubeEnv(credentialsId: 'sonar-api') {
        sh 'mvn clean package sonar:sonar'
}


}

}
}

stage('Quality Gate Status') {

steps {

  script{

    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-api'

  }

}

}

stage('nexus repo creation') {

steps {

script {
def readPomVer = readMavenPom file: 'pom.xml'
def nexusRepo = readPomVer.version.endsWith("SNAPSHOT") ? "demoapp-snapshot" : "demoapp-relaease"
nexusArtifactUploader artifacts: 

    [
         [artifactId: 'springboot', 
         classifier: '', 
         file: 'target/Uber.jar', 
         type: 'jar'
         ]
    ], 
         credentialsId: 'nexuspass', 
         groupId: 'com.example', 
         nexusUrl: '15.206.185.162:8081', 
         nexusVersion: 'nexus3', 
         protocol: 'http', 
         repository: nexusRepo, 
         version: "${readPomVer.version}"

    
}

}

}

stage('Docker image building') {
  steps {

    script{

    sh 'docker image build -t $JOB_NAME:$BUILD_ID .'
    sh 'docker image tag $JOB_NAME:$BUILD_ID jojotesthub/$JOB_NAME:$BUILD_ID'
  }   
}

}

stage('push to docker hub') {
steps {

script {

    withCredentials([string(credentialsId: 'docker_cred', variable: 'docker_hub_cred')]) {
    sh 'docker login -u jojotesthub -p $docker_hub_cred'
    sh 'docker image push jojotesthub/$JOB_NAME:$BUILD_ID'
}
}

}

}

}

}