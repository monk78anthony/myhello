pipeline {
   agent any
   environment {
       registry = "monk78anthony/myhello"
       GOCACHE = "/tmp"
   }
   stages {
       stage('Build') {
           agent {
               docker {
                   image 'golang'
               }
           }
           steps {
               // Create our project directory.
               sh 'cd ${GOPATH}/src'
               sh 'mkdir -p ${GOPATH}/src/hello-world'
               // Copy all files in our Jenkins workspace to our project directory.
               sh 'cp -r ${WORKSPACE}/* ${GOPATH}/src/hello-world'
               // Build the app.
               sh 'go build'
           }
       }
       stage('Test') {
           agent {
               docker {
                   image 'golang'
               }
           }
           steps {
               // Create our project directory.
               sh 'cd ${GOPATH}/src'
               sh 'mkdir -p ${GOPATH}/src/hello-world'
               // Copy all files in our Jenkins workspace to our project directory.
               sh 'cp -r ${WORKSPACE}/* ${GOPATH}/src/hello-world'
               // Remove cached test results.
               sh 'go clean -cache'
               // Run Unit Tests.
               sh 'go test ./*_test.go -v -short'
           }
       }
       stage('Code Quality Check via SonarQube') {
       steps {
           script {
           def scannerHome = tool 'sonarqube';
               withSonarQubeEnv('sonar') {
               sh "${tool("sonarqube")}/bin/sonar-scanner \
               -Dsonar.projectKey=myhello \
               -Dsonar.sources=. \
               -Dsonar.host.url=http://3.238.135.249:9000 \
               -Dsonar.login=3e2f31c0a709529ea6a86c4ae977a052b5a3fa38"
                   }
               }
           }
       }
       stage('Publish') {
           environment {
               registryCredential = 'dockerhub'
           }
           steps{
               script {
                   def appimage = docker.build registry + ":$BUILD_NUMBER"
                   docker.withRegistry( '', registryCredential ) {
                       appimage.push()
                       appimage.push('latest')
                   }
               }
           }
       }
       stage('Docker Remove Image') {
         steps {
           sh "docker rmi monk78anthony/myhello:${env.BUILD_NUMBER}"
         }
       }
       stage ('Deploy') {
           steps {
               withKubeConfig([credentialsId: 'kubeconfig']) {
                   sh '/usr/local/bin/kubectl delete deploy hello-deployment'
                   sh '/usr/local/bin/kubectl apply -f /usr/local/bin/service.yaml'
                   sh 'cat /usr/local/bin/deployment.yaml | sed "s/{{BUILD_NUMBER}}/$BUILD_NUMBER/g" | /usr/local/bin/kubectl apply -f -'
               }
           }
       }
   }
}
