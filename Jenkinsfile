pipeline {
    agent {
        label "jenkins-jx-base"
    }
    environment {
      DOCKER_REGISTRY = 'docker.artifactory.liatr.io'
    }
    stages {
        stage('Build') {
            steps {
                skaffoldBuild()
            }
        }
        stage('Deploy to Local Context') {
            steps {
                deployLocalContext()
            }
        }
        stage('Functional Test') {
            steps {
                functionalTest()
            }
        }
    }
}

def skaffoldBuild() {
    container('jx-base') {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'sonarqubeToken')]) {
            sh "echo 'sonar.login=${sonarqubeToken}' >> sonar.properties"
        }
        docker.withRegistry("https://${DOCKER_REGISTRY}", 'artifactory-credentials') {
            sh "skaffold build -p jenkins"
        }
    }
}

def deployLocalContext(){
    container('jx-base'){
        sh "skaffold run"
    }
}

def functionalTest(){
    container('maven') {
        sh 'kubectl port-forward $(kubectl get pods | grep "knowledge-share-app" | cut -d " " -f1 | head -n1) 8080:8080 &'
        sh 'cd functional-tests && mvn clean test -DappUrl=http://localhost:8080'
        sh 'pkill -f "port-forward knowledge-share-app"'
    }
}
