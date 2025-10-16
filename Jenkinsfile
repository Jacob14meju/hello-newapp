def appname = "hello-newapp"
def repo = "elevy99927"  // Replace with your DockerHub username
def artifactory = "docker.io" 
def appimage = "docker.io/${repo}/${appname}"
def apptag = "${env.BUILD_NUMBER}"
def DEPLOY = true

podTemplate(containers: [
      containerTemplate(name: 'jnlp', image: 'jenkins/inbound-agent', ttyEnabled: true),
      containerTemplate(name: 'docker', image: 'gcr.io/kaniko-project/executor:debug-v0.19.0', command: "/busybox/cat", ttyEnabled: true)
  ],
  volumes: [
     configMapVolume(mountPath: '/kaniko/.docker/', configMapName: 'docker-cred')
  ])  {
    node(POD_LABEL) {
        stage('checkout') {
            container('jnlp') {
            sh '/usr/bin/git config --global http.sslVerify false'
	    checkout scm
          }
        } // end chackout

        stage('build') {
            container('docker') {
              echo "Building docker image..."
              sh "/kaniko/executor --force --context . --dockerfile Dockerfile --destination ${appimage}:${apptag} --cache=true"
            }
        } //end build
        stage('deploy') {
              container('docker') {
	              if (DEPLOY) {
                        echo "***** Doing some deployment stuff *********"
                        sh """
                          kubectl set image deployment/hello-newapp \
                          hello-newapp=${appimage}:${apptag} \
                          --namespace default
                    """
                    }  else {
                        echo "***** NO DEPLOY - Doing somthing else. Testing? *********"
                    }
                  }
                } //end deploy
                } // end node
} // end podTemplate