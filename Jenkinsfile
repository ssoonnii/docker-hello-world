podTemplate(label: 'docker-build', 
  containers: [
    containerTemplate(
      name: 'git',
      image: 'alpine/git',
      command: 'cat',
      ttyEnabled: true
    ),
    containerTemplate(
      name: 'docker',
      image: 'docker',
      command: 'cat',
      ttyEnabled: true
    ),
  ],
  volumes: [ 
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'), 
    secretVolume(secretName: 'jenkins-tls', mountPath: '/certs')
  ],
  envVars: [
    envVar(key: 'JAVA_OPTS', value: '-Djavax.net.ssl.trustStore=/certs/tls.crt -Djavax.net.ssl.trustStorePassword=changeit')
  ]
) {
    node('docker-build') {
        def harborCred = credentials('harbor_credential')
        def appImage
        
        stage('Checkout'){
            container('git'){
                checkout scm
            }
        }
        
        stage('Build'){
            container('docker'){
                script {
                    appImage = docker.build("hyundai-harbor.tanzu.lab/app/node-hello-world:${env.BUILD_NUMBER}")
                }
            }
        }
        
        stage('Test'){
            container('docker'){
                script {
                    appImage.inside {
                        sh 'npm install'
                        sh 'npm test'
                    }
                }
            }
        }

        stage('Push'){
            container('docker'){
                script {
                    docker.withRegistry('https://hyundai-harbor.tanzu.lab', harborCred){
                        appImage.push("${env.BUILD_NUMBER}")
                        appImage.push("latest")
                    }
                }
            }
        }
    }
    
}
