# Setup ArgoCD on Kubernetes Cluster


## Step 1: Install ArgoCD

Run the following commands to install Argocd 

```
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

```
## Step 2: Edit ArgoCD service to run as NodePort

Edit the service by running the following command 

  kubectl -n argocd edit svc argocd-server

![](Images/b30.png)

Now check that the service is now using NodePort 

![](Images/b28.png)

Check the port on which service is running and then check on browser with nodeip:port 

![](Images/b31.png)

## Step 3: Configure ArgoCD

After this check the password for login in Argocd by running the following command 

  kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

![](Images/b29.png)

Enter username as admin and take password from the cli as follows 

![](Images/b32.png)

Now you'll see the following Argocd screen after login 

![](Images/b33.png)

Before starting with the deployment on Argocd, you will need to create a git repository & push the Helm charts to the repository 

## Step 4: Add deployment stage in Jenkins script 

Update the script on the Jenkins console and add the deployment stage where you need to git repository url and credentials

```
pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: maven
            image: maven:alpine
            command:
            - cat
            tty: true
          - name: docker
            image: docker:latest
            command:
            - cat
            tty: true
            volumeMounts:
             - mountPath: /var/run/docker.sock
               name: docker-sock
          volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock    
        '''
    }
  }
  stages {
    stage('Clone Repository') {
      steps {
        container('maven') {
        git(
            url: "https://github.com/sq-ldc/sentiment-analyzer-example-app.git",
            credentialsId: 'git-user',
            branch: "main"
          )
        //      git branch: 'master', changelog: false, poll: false, url: 'https://github.com/sq-ldc/sentiment-analyzer-example-app.git'
        // }
      }
      }
    }  
    stage('Login Into Docker') {
      steps {
        container('docker') {
          sh 'docker login --username xxxx --password xxx'
      }
    }
    }
    stage('Build Docker Images') {
      steps {
        container('docker') {
          dir("sa-frontend") {
            sh 'docker build -t reponame/sentiment-analysis-frontend:$BUILD_NUMBER .'
            sh 'docker build -t reponame/sentiment-analysis-web-app:$BUILD_NUMBER .'
          }
          dir("sa-logic") {
            sh 'docker build -t reponame/sentiment-analysis-logic:$BUILD_NUMBER .'
          }
        }
      }
    }
     stage('Push Images Docker to DockerHub') {
      steps {
        container('docker') {
          sh 'docker push reponame/sentiment-analysis-frontend:$BUILD_NUMBER'
          sh 'docker push reponame/sentiment-analysis-web-app:$BUILD_NUMBER'
          sh 'docker push reponame/sentiment-analysis-logic:$BUILD_NUMBER'
      }
     }
     }
  
      stage('Deploy') {
      steps {
          git(
            url: "https://github.com/sq-ldc/sentiment-analyzer-example-app.git",
            credentialsId: 'git-user',
            branch: "main"
          )
        withCredentials([usernamePassword(credentialsId: 'git-user', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
         container('maven') {
           script {
            last_started_build_stage = env.STAGE_NAME
              sh '''
              git config --global user.email abc@squareops.com
              git config --global user.name password123
              git config --global --add safe.directory /home/jenkins/agent/workspace/${JOB_NAME}
              cd sa-frontend
              yq e -i '(.image.tag = "'${BUILD_NUMBER}'")' values.yaml
              cd ../sa-logic
              yq e -i '(.image.tag = "'${BUILD_NUMBER}'")' values.yaml
              cd ../sa-webapp
              yq e -i '(.image.tag = "'${BUILD_NUMBER}'")' values.yaml
              git add .
              git commit -m 'Docker Image version Update "'$JOB_NAME'"-"'$BUILD_NUMBER'"'
              git push https://${USERNAME}:${PASSWORD}@github.com/sq-ldc/sentiment-analyzer-example-app.git           
              '''
        }
      }
    }
   }
    post {
      always {
        container('docker') {
          sh 'docker logout'
      }
      }
    }
}
  }
}
```
## Step 5: Create application in ArgoCD

Firstly connect the repo, so go to settings option 

![](Images/b34.png)

choose connection type HTTPS 

![](Images/b35.png)

click on **new app** and enter the following information 

![](Images/b36.png)

![](Images/b37.png)

![](Images/b38.png)

![](Images/b39.png)

![](Images/b40.png)

![](Images/b41.png)

similarly create application for sa-webapp and sa-frontend 

![](Images/b41.png)