pipeline {
    agent any
    stages {
        stage('Git Clone') {
            steps {
                script {
                    try {
                        git url: "git@github.com:KSLH/ECS-JENKINS-PIPELINE.git:", branch: "*/main", credentialsId: "jenkins"
                        sh "sudo rm -rf ./.git"
                        env.cloneResult=true
                    } catch (error) {
                        print(error)
                        env.cloneResult=false
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }
        stage('build-docker-image') {
            agent { label 'default' }
            steps {
                sh 'docker build -t ${REGISTRY}:${BUILD_NUMBER} -t ${REGISTRY}:latest .'
                }
                }
                
        stage('push-ecr') {
    agent { label 'default' }
    steps {
        withAWS(role: ASSUME_ROLE, roleAccount: ACCOUNT, externalId:'externalId') {
            sh 'aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${REGISTRY}'
            sh 'docker push ${REGISTRY}:${BUILD_NUMBER}'
            sh 'docker push ${REGISTRY}:latest'
            sh 'docker rmi ${REGISTRY}:${BUILD_NUMBER}'
            sh 'docker rmi ${REGISTRY}:latest'
        }
    }
}
        stage('deploy-ecs') {
    agent { label 'default' }
    steps {
        withAWS(role: ASSUME_ROLE, roleAccount: ACCOUNT, externalId:'externalId') {
            sh 'aws ecs update-service --force-new-deployment --cluster ${CLUSTER} --service ${SERVICE} --region ${REGION}'
            sh 'echo "https://${REGION}.${CONSOLE}/ecs/v2/clusters/${CLUSTER}/services/${SERVICE}/health?region=${REGION}"'
        }
    }
}
    }
