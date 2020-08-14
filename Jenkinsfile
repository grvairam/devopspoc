pipeline {

    agent any
    
       stages 
       {
           stage("Workspace Cleanup")
           {
            steps {
                 dir("${WORKSPACE}")
                {
                    deleteDir()
                }
                dir("${WORKSPACE}@tmp")
                {
                    deleteDir()
                }
              echo "clean up completed"
              //This will print the env varialbe. Just to print here
              echo "PATH =${PATH}"
            }
        }

          stage("Code Checkout") 
          {
            steps {
                echo "checking out the code"
                git GIT_URL
               	// This is hardcoding 
               //git url: "http://172.18.2.18/Vairam/devopspoc.git"
            }
        }
        
        stage("Code Build")
        {
            steps {
                echo "Building out the code"
                sh "mvn package"
                  }
        }
        
           stage("Dynamaically Replacing the  Image tag version ")
        {
            steps {
                echo "Replacing the tag version"
                sh 'sed "s/imagetagversion/${BUILD_ID}/g" "${WORKSPACE}/tomcat-pod.yaml" > "${WORKSPACE}/tomcat-pod-changed.yaml"'
                sh 'rm "${WORKSPACE}/tomcat-pod.yaml"'
                sh 'mv "${WORKSPACE}/tomcat-pod-changed.yaml" "${WORKSPACE}/tomcat-pod.yaml"'
                
                sh 'sed "s/imagetagversion/${BUILD_ID}/g" "${WORKSPACE}/tomcat-deployment.yaml" > "${WORKSPACE}/tomcat-deployment-changed.yaml"'
                sh 'rm "${WORKSPACE}/tomcat-deployment.yaml"'
                sh 'mv "${WORKSPACE}/tomcat-deployment-changed.yaml" "${WORKSPACE}/tomcat-deployment.yaml"'
                
                  }
        }
        
        
        
         stage("Docker Image build")
        {
            steps {
                echo "Building docker image"
                echo "$PATH"
                 sh "docker build -t vairam86/demodocker:${BUILD_ID} ."
                  }
        }
        
           stage("Docker Image push")
        {
            steps {
                echo "pushing  the docker image"
                echo "$PATH"
                
                withCredentials([string(credentialsId: 'Dockerpwd', variable: 'dockerhubPwd')]) {
                    sh "docker login -u vairam86 -p ${dockerhubPwd}"
}
                 sh "docker push vairam86/demodocker:${BUILD_ID}"
                  }
        }
        
        //  stage("Run Docker images")
        // {
        //     steps {
        //         echo "Running docker image"
        //          sh "docker run -d -p 8083:8080 vairam86/demodocker:${BUILD_ID}"
        //           }
        // }
        
          stage("Kubernetes deployment")
        {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubefile', namespace: 'default', serverUrl: 'https://192.168.99.101:8443') {
                 
                 script{
                     try{
                         //sh "kubectl version"
                       // sh "kubectl get pods"
                         sh "kubectl create -f tomcat-deployment.yaml"
                         sh "kubectl create -f tomcat-services.yaml"
                         // sh "kubectl create -f tomcat-services2.yaml"
                     }
                     catch(error){
                        // sh "kubectl version"
                      //  sh "kubectl get pods"
                      sh "kubectl apply -f tomcat-deployment.yaml"
                      sh "kubectl apply -f tomcat-services.yaml"  
                     }
                 }
                     
                 }
                
            }
        }
        
    }
        
}
