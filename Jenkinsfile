pipeline {
   agent any
   environment {
       registry = "monk78anthony/myhello"
       GOCACHE = "/tmp"
   }
   stages {
       stage('Build the Go binaries on a Container') {
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
       stage('Test the Go binaries on a Container') {
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
           def scannerHome = tool 'sonarqube-scanner';
               withSonarQubeEnv('sonar') {
               sh "${tool("sonarqube-scanner")}/bin/sonar-scanner \
               -Dsonar.projectKey=myhello \
               -Dsonar.sources=. \
               -Dsonar.host.url=http://34.204.185.219:9000 \
               -Dsonar.login=3e2f31c0a709529ea6a86c4ae977a052b5a3fa38"
                   }
               }
           }
       }
       stage('Publish the image to Docker Registry') {
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
       stage('Docker Remove Image Locally') {
         steps {
           sh "docker rmi monk78anthony/myhello:${env.BUILD_NUMBER}"
         }
       }
       stage ('Deploy and Expose on Kubernetes') {
           steps {
               withKubeConfig([credentialsId: 'kubeconfig']) {  
                   sh '/usr/local/bin/kubectl apply -f /usr/local/bin/service.yml'
                   sh 'cat /usr/local/bin/deployment.yml | sed "s/{{BUILD_NUMBER}}/$BUILD_NUMBER/g" | /usr/local/bin/kubectl apply -f -'
               }
           }
       }
       stage('ArgoCD Update'){
         steps{
           sh 'cat /home/ec2-user/myhello/argocd/deployment.yml | sed "s/{{BUILD_NUMBER}}/$BUILD_NUMBER/g"
         }
      }  
   }
}
