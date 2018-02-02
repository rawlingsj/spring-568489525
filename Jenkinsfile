
pipeline {
  agent {
    label "jenkins-maven"
  }

  environment {
    ORG 		= 'jenkinsx'
    APP_NAME    = 'spring-568489525'
  }

  stages {

    stage('Build Release') {
      steps {
        container('maven') {
          // ensure we're not on a detached head
          sh "git checkout master"

          // until we switch to the new kubernetes / jenkins credential implementation use git credentials store
          sh "git config credential.helper store"

          // so we can retrieve the version in later steps
          sh "echo \$(jx-release-version) > VERSION"
          sh "mvn versions:set -DnewVersion=\$(jx-release-version)"
        }

        dir ('./charts/spring-568489525') {
          container('maven') {
            sh "make tag"
          }
        }

        container('maven') {
          sh 'mvn clean deploy'
          sh "docker build -f Dockerfile.release -t $JENKINS_X_DOCKER_REGISTRY_SERVICE_HOST:$JENKINS_X_DOCKER_REGISTRY_SERVICE_PORT/$ORG/$APP_NAME:\$(cat VERSION) ."
          sh "docker push $JENKINS_X_DOCKER_REGISTRY_SERVICE_HOST:$JENKINS_X_DOCKER_REGISTRY_SERVICE_PORT/$ORG/$APP_NAME:\$(cat VERSION)"
        }
      }
    }
    stage('Deploy Staging') {

      steps {
        dir ('./charts/spring-568489525') {
          container('maven') {

            sh 'make release'
            sh 'helm install . --namespace staging --name example-release'
            sh 'exposecontroller --namespace staging --http' // until we switch to git environments where helm hooks will expose services
          }
        }
      }
    }
  }
}
