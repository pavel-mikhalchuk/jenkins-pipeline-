pipeline {
  agent { label 'kubectl-helm' }
  options { 
    buildDiscarder(logRotator(numToKeepStr: '5'))
    timestamps () 
  }
  parameters { 
    choice(name: 'NAMESPACE', choices: ['dev-dev', 'tst-test'], description: 'Kubernetes Namespace') 
    string(name: 'GIT_BRANCH', defaultValue: 'develop', description: 'Git Branch')
    string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker Image Tag to deploy')
    text(name: 'RESOURCES', 
      defaultValue: 

'''resources:
  resources:
    requests:
      memory: "2Gi"
      cpu: "1"
    limits:
      memory: "3Gi"
      cpu: "2"
javaOpts: "-Xms1g -Xmx3g -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.local.only=false -Dcom.sun.management.jmxremote.port=9014 -Dcom.sun.management.jmxremote.rmi.port=9014 -Djava.rmi.server.hostname=127.0.0.1"''', 
      
      description: 'Kubernetes POD resources requests and limits + JavaOpts')
  }
  environment {
    APP_NAME='filing'
    IMAGE="${APP_NAME}:${IMAGE_TAG ?: 'latest'}"
    HOST_NAME="${APP_NAME}.${hostByNs(NAMESPACE)}.in.in.alutech24.com"
    HELM_RELEASE="${APP_NAME}-${NAMESPACE}"
  }
  stages {
    stage('git repo checkout') {
      steps {
        git branch: '${GIT_BRANCH}', 
          credentialsId: '4f1ed3a6-d697-495c-a719-51b33e2c807c', 
          url: 'http://bb.alutech-mc.com:8080/scm/as/filing.git'
      }
    }
    stage('pick namespace specific app config') {
      steps {
        sh "cp src/main/resources/application.${NAMESPACE}.properties kubernetes/helm-chart/filing/application.properties"
      }
    }
    stage('define helm chart parameters') {
      steps {
        writeFile file: 'kubernetes/helm-chart/filing/values.yaml',
          text: 

"""replicaCount: 1
gitBranch: ${GIT_BRANCH}
image:
  repository: ${DOCKER_REPO}
  tag: ${IMAGE}
  pullPolicy: IfNotPresent
service:
  externalPort: 80
  internalPort: 8080
jenkinsBuildNumber: ${JOB_NAME}-${BUILD_NUMBER}
host: ${HOST_NAME}
${RESOURCES}"""

      }
    }
    stage('install helm chart') {
      steps {
        container('kubectl-helm') {
          sh "helm upgrade --namespace ${NAMESPACE} ${HELM_RELEASE} -i kubernetes/helm-chart/filing --dry-run --debug"
          sh "helm upgrade --namespace ${NAMESPACE} ${HELM_RELEASE} -i kubernetes/helm-chart/filing"
        }
      }
    }
  }
}

def hostByNs(ns) {
    return ns.contains('-') ? "${ns.split('-')[1]}.${ns.split('-')[0]}" : ns
}
