podTemplate(label: 'mypod', containers: [
    containerTemplate(name: 'docker', image: 'docker', ttyEnabled: true, command: 'cat'),
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
            //OpenShift docker registry
            //imageName = "172.30.1.1:5000/openshift/sample-demo"+":latest"

            //Kubernetes docker registry
            imageName = "localhost:5000/sample-demo"+":latest"
            env.BUILDIMG=imageName
        } 

        stage ("Build war"){
            container('maven') {
                echo "Building version"
                sh 'mvn -B clean install -Dmaven.test.skip=true -Dfindbugs.fork=false'

                //sh 'mvn -B clean install -s nexus_settings.xml -Dmaven.test.skip=true -Dfindbugs.fork=false'
            }
        }

        // stage ("Unit Tests"){
        //     container('maven') {
        //         echo "Unit Tests"
        //         sh 'mvn -B  -s nexus_settings.xml test'
        //     }
        // }

        stage ("Docker Build"){
            container('docker') {
                //Docker login for minishift docker registry, comment out for minikube
                //sh "docker login -u admin -p pbWZvJEl5BrE90Nfm19RGRiUJ8_BUvyYm5Y2eHivpcM 172.30.1.1:5000"
                
                sh "docker build -t ${imageName} ."
                sh "docker push ${imageName}"
            }  
        }

        stage ("Deploy with Helm"){
            container('helm') {
                sh "helm upgrade --install demo-sample ./demo-sample"
            }
        }
    }
}
