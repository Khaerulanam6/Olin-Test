def project = ""
def appName = ""
def namespace = ""
def proxyType = ""
def proxyAddress = ""
def proxyPort = ""
def token = ""
def chatID = ""

pipeline {
  agent {
    kubernetes {
      defaultContainer 'jnlp'
      yaml """
        apiVersion: v1
        kind: Pod
        metadata:
        labels:
          component: ci
        spec:
          tolerations:
          - key: "jenkins"
            operator: "Equal"
            value: "agent"
            effect: "NoSchedule"
          affinity:
            nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
               nodeSelectorTerms:
                -  matchExpressions:
                   - key: jenkins
                     operator: In
                     values:
                     - agent
          containers:
          - name: gcloud
            image: google/cloud-sdk:263.0.0-alpine
            imagePullPolicy: IfNotPresent
            command:
            - cat
            tty: true
        """
    }
  }
  
  stages {
    stage("build image") {
      environment {
        IMAGE_REPO = "gcr.io/${project}/${appName}"
        IMAGE_TAG = "${env.GIT_COMMIT.substring(0,7)}"
      }  
      steps {
        container("gcloud") {
        withCredentials([file(credentialsId: "xxxxx", variable: "JSONKEY")]) {
            sh "gcloud config set proxy/type ${proxyType}"
            sh "gcloud config set proxy/address ${proxyAddress}"
            sh "gcloud config set proxy/port ${proxyPort}"

            sh "cat ${JSONKEY} >> key.json"
            sh "gcloud auth activate-service-account --key-file=key.json"
            sh "gcloud builds submit --project ${project} --tag ${IMAGE_REPO}:${IMAGE_TAG} ."
          }
        }
      }
    }

stage ('Deployment') {
  parallel {
    stage("Deploy to k8s") {
      when {
        branch 'xxxxx'
      }
      environment {
        IMAGE_REPO = "gcr.io/${project}/${appName}"
        IMAGE_TAG = "${env.GIT_COMMIT.substring(0,7)}"
      }
      steps {
        container("helm") {
        withCredentials([file(credentialsId: "xxxxxx", variable: "KUBECONFIG")]) {
            sh "mkdir -p ~/.kube/"
            sh "cat ${KUBECONFIG} >> ~/.kube/config"
            
            sh """
              helm upgrade ${appName} ./helm/${appName} \
                --set-string image.repository=${IMAGE_REPO},image.tag=${IMAGE_TAG} \
                -f ./helm/${appName}/values.yaml --debug --install --namespace ${namespace}
            """
                }
              }
            }
          }
      }
    }
  }
}