pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_DEFAULT_REGION    = 'ap-south-1'
    }
	
    stages {
        stage('SonarQube Analysis') {
            agent {
                label 'sonar'
            }
            steps{
                withSonarQubeEnv('sonarCodeSS') { 
                    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=sonardemo"
                 }
                }
            }
        stage('Build Stage') {
            agent {
                label 'jfrog'
            }
            steps {
                sh 'mvn clean install'
            }
            post {
                success {
                    script {
                        def server = Artifactory.newServer(url: 'http://13.126.88.138:8081/', credentialsId: 'jfrog')
                        def rtMaven = Artifactory.newMavenBuild()
                        rtMaven.deployer server: server, releaseRepo: 'libs-release/', snapshotRepo: 'libs-snapshot/'
                        rtMaven.tool = 'maven'
                        rtMaven.run(pom: 'pom.xml', goals: 'clean install')
                    }
                }
            }
        }
	stage('Push to ECR and Deply to eks') {
            agent {
                label 'deploy'
            }
            steps {
                script {
                    sh 'curl -o jenkins-test-2.0.jar http://13.126.88.138:8081/artifactory/libs-release/com/example/jenkins-test/2.0/jenkins-test-2.0.jar'
                    sh '''
                        aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin 804480554088.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com
                        docker build -t sstest .
                        docker tag sstest:latest 804480554088.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/sstest:latest
                        docker push 804480554088.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/sstest:latest
                        aws eks update-kubeconfig --region ap-south-1 --name cluster-eksctl
			kubectl version --client
                        kubectl apply -f test.yaml
                        sleep 10
                        kubectl get pods -n ss-dev
                        kubectl get svc -n ss-dev
                    '''
                }
            }
        }
}
}
