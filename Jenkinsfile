podTemplate(label: 'mypod', containers: [
    containerTemplate(name: 'docker', image: 'docker', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.0', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:latest', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'maven', image: 'maven:3.5.0-jdk-8-alpine', ttyEnabled: true, command: 'cat',
        resourceRequestCpu: '100m',
        resourceLimitMemory: '1200Mi')
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
  ]) {
    node('mypod') {

        stage ("Checkout Source"){
            checkout scm
            appName = "sample-demo"
            imageName = "localhost:5000/sample-demo"+":latest"
            env.BUILDIMG=imageName
        } 

        stage ("Build war"){
            container('maven') {
                echo "Building version"
                sh 'mvn -B clean install -Dmaven.test.skip=true -Dfindbugs.fork=false'
            }
        }

        // stage ("Unit Tests"){
        //     container('maven') {
        //         echo "Unit Tests"
        //         sh 'mvn -B test -DconnectorHost=0.0.0.0'
        //     }
        // }

        stage ("Docker Build"){
            container('docker') {
                sh "docker build -t ${imageName} ."
                sh "docker push ${imageName}"
            }  
        }

        stage ("Create helm chart"){
            container('helm') {
                sh "helm install --name demo-sample demo-sample"
            }
        }
    }
}
