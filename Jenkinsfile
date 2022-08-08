pipeline {
  agent {
    kubernetes {
      cloud 'k8s115'
      label 'slave'
      inheritFrom 'jenkins-slave'
     // defaultContainer 'node'
      yaml """\
apiVersion:
kind: Pod
metadata:
  labels:
    jenkins: "slave"
    jenkins/label: 'k8s-slave'
spec:
  containers:
  - command:
    - "cat"
    image: "node:alpine"
    imagePullPolicy: "IfNotPresent"
    name: "node"
    tty: true
  - command:
    - "cat"
    image: "cnych/kubectl"
    imagePullPolicy: "IfNotPresent"
    name: "kubectl"
    tty: true
    volumeMounts:
    - mountPath: "/home/jenkins/.kube"
      name: "volume-0"
      readOnly: false
    - mountPath: "/var/run/docker.sock"
      name: "volume-1"
      readOnly: false
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  - command:
    - "cat"
    image: "library/tool:v3"
    imagePullPolicy: "IfNotPresent"
    name: "tools"
    tty: true
  - command:
    - "cat"
    image: "docker"
    imagePullPolicy: "IfNotPresent"
    name: "docker"
    resources:
      limits: {}
      requests: {}
    tty: true
    volumeMounts:
    - mountPath: "/home/jenkins/.kube"
      name: "volume-0"
      readOnly: false
    - mountPath: "/var/run/docker.sock"
      name: "volume-1"
      readOnly: false
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  volumes:
  - hostPath:
      path: "/root/.kube"
    name: "volume-0"
  - hostPath:
      path: "/var/run/docker.sock"
    name: "volume-1"
  - emptyDir:
      medium: ""
    name: "workspace-volume"
      """.stripIndent()

    }
  }

	options {
        timeout(time: 20, unit: 'MINUTES')
		//gitLabConnection('gitlab')
	}
    stages {
        stage('checkout') {
            steps {
                container('tools') {
                    checkout scm
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    container('tools'){
                        switch(env.comp){
                            case "qfcsspiders":
                                env.testDir = "qfcsspiders"
                                break
                            case "assets-cms":
                                env.testDir = "assets-cms"
                                break
                            default:
                                env.testDir = "all"
                                break
                        }
                        sh 'robot -d artifacts/ ${testDir}/*'
                        step([
                            $class : 'RobotPublisher',
                            outputPath: 'artifacts/',
                            outputFileName : "output.xml",
                            disableArchiveOutput : false,
                            passThreshold : 100,
                            unstableThreshold: 80.0,
                            onlyCritical : true,
                            otherFiles : "*.png"
                        ])
                        archiveArtifacts artifacts: 'artifacts/*', fingerprint: true
                    }
                }
            }
        }
    }
}