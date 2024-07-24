podTemplate(label: 'docker-build', 
  containers: [
    containerTemplate(
      name: 'jnlp',
      image: 'jenkins/inbound-agent',
      args: '${computer.jnlpmac} ${computer.name} -noCertificateCheck',  // -noCertificateCheck 옵션 추가
      ttyEnabled: true
    ),
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
