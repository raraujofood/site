node {
  def project = 'jenkins'
  def appName = 'images'
  def feSvcName = "${appName}-frontend"
  def imageTag = "${project}/${appName}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"

  checkout scm

  stage 'Build image'
  sh("docker build -t ${imageTag} .")

  stage 'Run Go tests'
  sh("docker run ${imageTag} go test")

//  stage 'Push image to registry'
//  sh("gcloud docker -- push ${imageTag}")

  stage "Deploy Application"
  switch (env.BRANCH_NAME) {
    // Roll out to staging environment
    case "staging":
        // Change deployed image in staging to the one we just built
        sh("sed -i.bak 's#nginx:latest#${imageTag}#' staging/*.yaml")
        sh("kubectl --namespace=staging apply -f jenkins/staging/services/")
        sh("kubectl --namespace=staging apply -f jenkins/staging/frontend/")
        sh("echo http://`kubectl --namespace=staging get service/${feSvcName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${feSvcName}")
        break
  }
}
