node {
  
  environment {
        DOCKER_CONTENT_TRUST = '1'
  }
  
  stage('Checkout') {
    checkout scm
  }

  stage('Build') {
    withCredentials([
        usernamePassword(credentialsId: 'docker-credentials',
                         usernameVariable: 'USERNAME',
                         passwordVariable: 'PASSWORD')]) {
      sh 'docker image build -t ${USERNAME}/demo-api:latest .'
    }
  }

  stage('Scan') {
    withCredentials([
        string(credentialsId: 'microscanner-token',
               variable: 'MICROSCANNER_TOKEN'),
        usernamePassword(credentialsId: 'docker-credentials',
                         usernameVariable: 'USERNAME',
                         passwordVariable: 'PASSWORD')]) {
      sh 'wget https://github.com/lukebond/microscanner-wrapper/raw/master/scan.sh -O /usr/local/bin/scan.sh && chmod +x /usr/local/bin/scan.sh'
      sh 'MICROSCANNER_OPTIONS=--continue-on-failure /usr/local/bin/scan.sh ${USERNAME}/demo-api:latest'
    }
  }

  stage('Push') {
    withCredentials([
        usernamePassword(credentialsId: 'docker-credentials',
                         usernameVariable: 'USERNAME',
                         passwordVariable: 'PASSWORD')]) {
      sh 'docker login -p "${PASSWORD}" -u "${USERNAME}"'
      sh 'docker image push ${USERNAME}/demo-api:latest'
    }
  }

  stage('Deploy') {
    withCredentials([
        file(credentialsId: 'kube-config',
             variable: 'KUBECONFIG')]) {
      sh 'kubectl apply -f deployment.yaml'
    }
  }
}
