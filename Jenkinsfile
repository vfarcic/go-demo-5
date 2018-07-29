// import java.text.SimpleDateFormat

// def props
// def label = "jenkins-slave-${UUID.randomUUID().toString()}"
// currentBuild.displayName = new SimpleDateFormat("yy.MM.dd").format(new Date()) + "-" + env.BUILD_NUMBER

// // TODO: namespace: "go-demo-5-build", // Not allowed with declarative
// // TODO: serviceAccount: "build",

pipeline {
  agent {
    go-demo-5 {
      label "my-pod"
      yamlFile "KubernetesPod.yaml"
    }      
  }
  environment {
    image = "vfarcic/go-demo-5"
    project = "go-demo-5"
    domain = "192.168.0.189.nip.io"
    cmAddr = "cm.192.168.0.189.nip.io"
  }
  stages {
    stage("build") {
      steps {
        container("docker") {
          k8sBuildImageBeta(image)
        }
      }
    }
    stage("func-test") {
      steps {
        container("helm") {
          k8sUpgradeBeta(project, domain, "--set replicaCount=2 --set dbReplicaCount=1")
        }
        container("kubectl") {
          k8sRolloutBeta(project)
        }
        container("golang") {
          k8sFuncTestGolang(project, domain)
        }
      }
      post {
        failure {
          container("helm") {
            k8sDeleteBeta(props.project)
          }
        }
      }
    }
    stage("release") {
      when {
          branch "master"
      }
      steps {
        container("docker") {
          k8sPushImage(image)
        }
        container("helm") {
          k8sPushHelm(project, "", cmAddr)
        }
      }
    }
  }
}
