node {

  def project = 'GCP-PROJECT'
  def appName = 'gceme'
  def feSvcName = "${appName}-frontend"
  def imageTag = "gcr.io/${project}/${appName}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"

  // Kube Config
  def gcloud = '/var/lib/jenkins/google-cloud-sdk/bin/gcloud'
  def kubectl = '/var/lib/jenkins/google-cloud-sdk/bin/kubectl'

  def cluster = 'CLUSTER-NAME'
  def zone = 'us-central1-b'

  checkout scm

  //stage 'Run Go tests'
  //sh("docker run ${imageTag} go test")

  stage('GET Kubernetes get-credentials') {
    // some block
    sh("${gcloud} container clusters get-credentials ${cluster} --zone ${zone}")
  }

  //stage 'GET cluster info'
  //sh("${gcloud} --configuration=default container clusters list")

  stage('GET cluster info') {
    // some block
    sh("${gcloud} --configuration=default container clusters list")
  }


  stage "Deploy Application"
  switch (env.BRANCH_NAME) {
    // Roll out to canary environment
    case "canary":

        //stage "Build docker image :: ${env.BRANCH_NAME} : ${imageTag}"
        stage("Build docker image :: ${env.BRANCH_NAME} : ${imageTag}"){
            sh("docker build -t ${imageTag} .")
        }

        //stage 'Push image to registry'
        stage('Push image to registry') {
            sh("${gcloud} docker -- push ${imageTag}")
        }

        // Change deployed image in canary to the one we just built
        sh("sed -i.bak 's#gcr.io/cloud-solutions-images/gceme:1.0.0#${imageTag}#' ./k8s/canary/*.yaml")

        stage("Kubernetes Deployment : ${env.BRANCH_NAME}"){
            sh("${kubectl} --namespace=production apply -f k8s/canary/")
        }

        break

    // Roll out to production
    case "master":

        /*echo "Build docker image :: ${env.BRANCH_NAME} : ${imageTag}"
        //sh("docker build -t ${imageTag} .")

        stage 'Push image to registry'
        sh("${gcloud} docker -- push ${imageTag}")
        */

        stage("Build docker image :: ${env.BRANCH_NAME} : ${imageTag}"){
            sh("docker build -t ${imageTag} .")
        }

        //stage 'Push image to registry'
        stage('Push image to registry') {
            sh("${gcloud} docker -- push ${imageTag}")
        }

        // Change deployed image in canary to the one we just built
        sh("sed -i.bak 's#gcr.io/cloud-solutions-images/gceme:1.0.0#${imageTag}#' ./k8s/production/*.yaml")

        stage("Kubernetes Deployment : ${env.BRANCH_NAME}"){
            sh("${kubectl} --namespace=production apply -f k8s/production/")
        }

        break

    // Roll out for other BRANCH_NAME
    default:

        echo "Deployment came for non-(canary | production)  : ${env.BRANCH_NAME}"
  }
}
