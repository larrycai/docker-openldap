podTemplate(label: 'mypod', containers: [
    containerTemplate(name: 'docker', image: 'docker:1.12.6-dind', 
        ttyEnabled: true, alwaysPullImage: true, privileged: true,
        envVars: [
            envVar(key: 'DOCKER_HOST', value: 'tcp://localhost:2375')
        ])
  ],
  volumes: [emptyDirVolume(memory: false, mountPath: '/var/lib/docker')]) {
  
  node ('mypod') {
    echo sh(returnStdout: true, script: 'env')
    
    container('docker') {
       stage ('Checkout') {
                checkout scm
                sh """
                pwd
                env
                """
       }
       stage ('Build a docker') {
                sh """
                docker version
                docker build .
                """
       }
}
       
