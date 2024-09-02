pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: buildah
                image: dtzar/helm-kubectl:latest
                command:
                - cat
                tty: true
                securityContext:
                  privileged: true
                volumeMounts:
                - name: buildah-storage
                  mountPath: /var/lib/containers
              volumes:
              - name: buildah-storage
                emptyDir: {}
            '''
        }
    }
    stages {
        stage('Install Git, Java, buildah, and podman dependencies') {
            steps {
                container('buildah') {
                    sh '''
                    apk add --no-cache git openjdk8 buildah coreutils
                    apk add --no-cache podman netavark aardvark-dns
                    '''
                }
            }
        }
        stage('Clone Git Repository') {
            steps {
                container('buildah') {
                    sh 'git clone https://github.com/kikan321/amazonlinux-httpd-Dockerfile.git'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                container('buildah') {
                    sh 'buildah bud -t my-docker-image:latest amazonlinux-httpd-Dockerfile/'
                }
            }
        }
        stage('Push Docker Image to Artifactory') {
            steps {
                container('buildah') {
                    withCredentials([usernamePassword(credentialsId: 'artifactory-credentials', usernameVariable: 'ARTIFACTORY_USER', passwordVariable: 'ARTIFACTORY_PASSWORD')]) {
                        sh '''
                        buildah login --tls-verify=false -u $ARTIFACTORY_USER -p $ARTIFACTORY_PASSWORD http://57.152.50.3:8082
                        buildah push --tls-verify=false my-docker-image:latest docker://57.152.50.3:8082/poc/my-docker-image:latest
                        '''
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                container('buildah') {
                    sh '''
                    cd amazonlinux-httpd-Dockerfile
                    kubectl apply -f deployment.yaml
                    '''
                }
            }
        }
    }
}
