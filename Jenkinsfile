
//TODO: Probably we should build one more jenkins slave with kubernetes inside &
// either hardcode credentials  of Dev,Prod,etc... cluster connections, or to pass KEYs and configs somehow, for instance by keeping in the Serivces Repos, or  we can have for every ENV it's jenkins-slave, i.e. slave-swarm-dind-kube-test-runcher && slave-swarm-dind-kube-gc-cluster1, probaly last

env.VERSION = env.BRANCH_NAME == 'master' ? 'latest' : env.BRANCH_NAME.tokenize('/').last()
def tag = params.latest ? 'latest' : env.BRANCH_NAME.tokenize('/').last()
env.DHUB_PROJECT = "test-jenkins"
env.DHUB_USER = "jenkins-test"
env.DHUB_PASS = "Dunerunner1"
env.SERVICE_NAME = "nginx-ai-release-1.2"

node('slave-swarm-dind') {
    timestamps {
        stage('Checkout') {
            echo 'Getting the code base...'
        checkout scm
        git url: 'https://github.com/gerasimhovhannisyan/test_build_jenkins.git', branch: env.BRANCH_NAME, credentialsId: 'd3a36e01-c34a-41b6-96f9-fbea0131315e'
        }

        try {
            stage('Build images') 
            dir('.') {
                echo 'Building docker images...'
                sh 'echo ${DHUB_PASS};'
                sh 'docker login harbor.picsart.tools -u ${DHUB_USER} -p ${DHUB_PASS}; echo "DockerLogin exitcode=${?}"'
                sh 'pwd; ls -al; docker build --tag=harbor.picsart.tools/${DHUB_PROJECT}/${SERVICE_NAME}:${VERSION} -f Dockerfile .; echo "Docker BUILD exitcode=${?}"'
                echo 'All images were built successfully!'
            }
            
             stage('Push images') {
                echo 'Pushing docker images...'
                sh 'docker push harbor.picsart.tools/${DHUB_PROJECT}/${SERVICE_NAME}:${VERSION}; echo "Docker PUSH exitcode=${?}"'
            }
       if (env.BRANCH_NAME =~ /release\/[0-9]+\.[0-9]+\.[0-9]+$/) {
                sshagent(credentials: ['ec011a52-2573-47b3-a627-ad7147e69613']) {
                    stage('Disable branch') {
                       echo 'Marking branch as built...'
                       sh 'git config --global user.email "jenkins@picsart.com"'
                       sh 'git config --global user.name "Jenkins"'
                       //sh 'git mv Jenkinsfile Jenkinsfile.disable && git commit -m "[auto commit] Disable pipeline after sucessful build" && git push origin ${BRANCH_NAME}'
                     }
                  }
             }

 
        } catch (err) {
            echo "Build failed: ${err}"
            currentBuild.result = 'FAILURE'
        }

        stage('Cleanup') {
            echo 'Removing containers...'
            sh 'docker rm -f $(docker ps -qa) 2>/dev/null || true'
            echo 'Removing images...'
            sh 'docker rmi -f $(docker images -qa) 2>/dev/null || true'
            echo 'Removing volumes...'
            sh 'docker volume rm $(docker volume ls -q) 2>/dev/null || true'
        }
    }
}

