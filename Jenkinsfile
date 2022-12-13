pipeline {

  agent {
    label 'maven'
  }

  stages {
    stage('Build') {
      steps {
        echo 'Building..'
        
        sh 'mvn clean package'
      }
    }
    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'
        
        script {

          openshift.withCluster() { 
  openshift.withProject("first-project") {
  
    def buildConfigExists = openshift.selector("bc", "imagebuild").exists() 
    
    if(!buildConfigExists){ 
      openshift.newBuild("--name=imagebuild", "--docker-image=registry.redhat.io/redhat-openjdk-18/openjdk18-openshift", "--binary") 
    } 
    
    openshift.selector("bc", "imagebuild").startBuild("--from-file=target/first-project-SNAPSHOT.jar", "--follow") } }

        }
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {

          openshift.withCluster() { 
            openshift.withProject("first-project") { 
              def deployment = openshift.selector("dc", "imagebuild") 
     
              if(!deployment.exists()){ 
                 openshift.newApp('imagebuild', "--as-deployment-config").narrow('svc').expose() 
                  } 
    
                      timeout(5) { 
                           openshift.selector("dc", "imagebuild").related('pods').untilEach(1) { 
                           return (it.object().status.phase == "Running") 
                                 } 
                            } 
                         } 
                      }

        }
      }
    }
  }
}
