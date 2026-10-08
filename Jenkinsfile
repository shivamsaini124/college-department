pipeline {

    agent any

    environment {
        DOCKER_IMAGE = "shivam3294/college-department"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Clone Code') {
            steps {
                echo '========================================'
                echo 'CLONING CODE FROM GITHUB'
                echo '========================================'

                git branch: 'main',
                    url: 'https://github.com/shivamsaini124/college-department.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '========================================'
                echo 'BUILDING DOCKER IMAGE'
                echo '========================================'

                sh '''
                    docker build --pull=false \
                        -t $DOCKER_IMAGE:$DOCKER_TAG .

                    docker tag \
                        $DOCKER_IMAGE:$DOCKER_TAG \
                        $DOCKER_IMAGE:latest
                '''
            }
        }

        stage('Push Image') {
            steps {

                echo '========================================'
                echo 'PUSHING IMAGE TO DOCKER HUB'
                echo '========================================'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )
                ]) {

                    sh '''
                        echo "$PASS" | docker login \
                            -u "$USER" \
                            --password-stdin

                        docker push $DOCKER_IMAGE:$DOCKER_TAG
                        docker push $DOCKER_IMAGE:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {

                echo '========================================'
                echo 'DEPLOYING TO KUBERNETES'
                echo '========================================'

                withCredentials([
                    file(
                        credentialsId: 'kuberconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {

                    sh '''
                        export KUBECONFIG="$KUBECONFIG"

                        echo "Checking Kubernetes cluster..."
                        kubectl config current-context

                        echo "Applying Kubernetes configuration..."
                        kubectl apply -f deployment.yaml

                        echo "Updating deployment image..."

                        kubectl set image \
                            deployment/college-department \
                            college-department=$DOCKER_IMAGE:$DOCKER_TAG

                        echo "Waiting for rollout..."

                        kubectl rollout status \
                            deployment/college-department

                        echo "========================================"
                        echo "DEPLOYMENT STATUS"
                        echo "========================================"

                        kubectl get deployment college-department

                        echo "========================================"
                        echo "POD STATUS"
                        echo "========================================"

                        kubectl get pods \
                            -l app=college-department

                        echo "========================================"
                        echo "SERVICE STATUS"
                        echo "========================================"

                        kubectl get service \
                            college-department-service

                        echo "========================================"
                        echo "IMAGE USED"
                        echo "========================================"

                        kubectl get deployment college-department \
                            -o jsonpath="{.spec.template.spec.containers[0].image}"

                        echo
                    '''
                }
            }
        }
    }

    post {

        success {
            echo '========================================'
            echo 'COLLEGE DEPARTMENT DEPLOYMENT SUCCESSFUL'
            echo '========================================'
            echo "Docker Image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
            echo 'Replicas: 3'
            echo 'NodePort: 30082'
            echo '========================================'
        }

        failure {
            echo '========================================'
            echo 'PIPELINE FAILED'
            echo '========================================'
        }
    }
}