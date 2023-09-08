// Uses Declarative syntax to run commands inside a container.
pipeline {
    agent {
        kubernetes {
            // Rather than inline YAML, in a multibranch Pipeline you could use: yamlFile 'jenkins-pod.yaml'
            // Or, to avoid YAML:
            // containerTemplate {
            //     name 'shell'
            //     image 'ubuntu'
            //     command 'sleep'
            //     args 'infinity'
            // }
            yaml '''
                apiVersion: v1
                kind: Pod
                metadata:
                  name: jnlp
                spec:
                  containers:
                  - name: shell
                    image: ubuntu
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
                  - name: git
                    image: alpine/git
                    command:
                    - /bin/cat
                    tty: true 
                  - name: kubectl
                    image: joshendriks/alpine-k8s
                    command:
                    - /bin/cat
                    tty: true    
                  volumes:
                    - name: docker-sock
                      hostPath:
                        path: /var/run/docker.sock
'''
        }
    }
    stages {
        stage('Chekout SCM') {
            steps {
                container('git') {
                    script {
                        sh '''
                            apt-get update -y && apt-get install git -y
                            git clone -b kaniko https://github.com/justmeandopensource/playjenkins.git
                            pwd
                            ls -al
                        '''
                    }
                }
                
            }
        }
        stage(' Build & Push Image') {
            environment {     
                DOCKERHUB_CREDENTIALS= credentials('dockerhubaccesstoken') 
            }
            steps {
                container('docker') {
                    sh '''
                    pwd
                    ls -al 
                    cd playjenkins
                    ls -al
                    docker build -t tarmizy/testing-deployment:$BUILD_NUMBER .
                    docker images
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push tarmizy/testing-deployment:$BUILD_NUMBER
                    docker image prune -a -f
                    
                    '''
                }
            }
        }
        stage('Deploy App to Kubernetes') {
            environment {     
                VERSION = "${env.BUILD_ID}"
            }
            steps {
                container('git') {
                  script{
                    withCredentials([usernamePassword(credentialsId: 'githubaccesstoken', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh '''
                          git clone -b main https://$GIT_USERNAME:$GIT_PASSWORD@github.com/tarmizy/demo-manifest.git
                          git config --global user.email 'jenkins@ci.cd'
                          ls -al
                          cd demo-manifest/enviroment-deploy
                          cat deployment.yaml
                          ls -al
                          versi=${BUILD_NUMBER}
                          echo $VERSION
                          sed -i "s/testing-deployment:.*/testing-deployment:${BUILD_NUMBER}/g" deployment.yaml
                          cat deployment.yaml
                          ls -al
                          cd ..
                          pwd
                          ls
                          git add .;git commit -m "add deploy";git push
                        '''                        
                    }
                  }
                }
            }
        }
        // stage('Deploy App to Kubernetes') {     
        //     steps {
        //         container('kubectl') {
        //             withCredentials([file(credentialsId: 'mykubeconfig', variable: 'KUBECONFIG')]) {
        //                 sh 'sed -i "s/tarmizy/testing-deployment:.*/tarmizy/testing-deployment:${BUILD_NUMBER}/" playjenkins/myweb.yaml'
        //                 sh 'kubectl apply -f myweb.yaml'
        //             }
        //         }
        //     }
        // }
  
    }
}
