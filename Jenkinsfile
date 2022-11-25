pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: docker
            image: docker:dind
            securityContext:
              allowPrivilegeEscalation: true
            command:
            - cat
            tty: true
            volumeMounts:
            - name: dockersock
              mountPath: 'var/run/docker.sock'
          - name: pods
            image: ravennaras/template:eksctl-argo-v1
            securityContext:
              allowPrivilegeEscalation: true
            tty: true
          volumes:
          - name: dockersock
            hostPath:
              path: /var/run/docker.sock
        '''
    }
  }
  environment{
    VERSION="${env.BUILD_ID}${GIT_COMMIT[0..2]}"
    DOCKERHUB_CREDENTIALS=credentials('docker')
    ARGOCD_CREDENTIALS=credentials('argocd')
    ARGOCD_URL=credentials('argocd-url')
  }
  stages {
    stage('BUILD') {
      steps {
        container('docker') {
            sh '''
              echo 'building deployment image'
              echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
              docker build -f ./src/backend/Dockerfile.be -t ravennaras/cilist-production-be:be-${VERSION} . --network host --no-cache --pull
              docker build -f ./src/frontend/Dockerfile.fe -t ravennaras/cilist-production-fe:fe-${VERSION} . --network host --no-cache --pull
            '''
        }
      }
    }
    stage('TEST') {
      steps {
        container('docker') {
            sh '''
              echo 'test something here'
              # docker run --network host aquasec/trivy image ravennaras/cilist-production-be:be-${VERSION} --security-checks vuln
              # docker run --network host aquasec/trivy image ravennaras/cilist-production-fe:fe-${VERSION} --security-checks vuln
              docker push ravennaras/cilist-production-be:be-${VERSION}
              docker push ravennaras/cilist-production-fe:fe-${VERSION}
            '''
        }
      }
    }
    stage('DEPLOY') {
      steps {
        container('pods'){
            sh '''
              echo 'deploy to cluster'
              argocd login $ARGOCD_URL --username $ARGOCD_CREDENTIALS_USR --password $ARGOCD_CREDENTIALS_PSW --insecure
	            echo argocd login successful
	            argocd app get cilist-production-be
		          argocd app get cilist-production-fe
              argocd app sync cilist-production-be
			        argocd app sync cilist-production-fe
            '''
        }
      }
    }
  }
}
