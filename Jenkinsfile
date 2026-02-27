pipeline {
    agent any

    environment {
        IMAGE_NAME = "2022bcs0208sanjana/wine_predict_2022bcs0208:latest"
        CONTAINER_NAME = "Lab7_mlops"
    }

    stages {

        stage('Pull Image') {
            steps {
                sh "docker pull $IMAGE_NAME"
            }
        }

        stage('Run Container') {
            steps {
                sh "docker run -d --name $CONTAINER_NAME -p 8000:8000 $IMAGE_NAME"
            }
        }

        stage('Wait for API') {
            steps {
                sh """
                for i in {1..10}; do
                  curl -s http://localhost:8000/health && exit 0
                  sleep 3
                done
                exit 1
                """
            }
        }

        stage('Valid Inference Test') {
            steps {
                sh """
                curl -s -X POST http://localhost:8000/predict \
                -H "Content-Type: application/json" \
                -d @valid_input.json
                """
            }
        }

        stage('Invalid Inference Test') {
            steps {
                sh """
                STATUS=\$(curl -s -o /dev/null -w "%{http_code}" \
                -X POST http://localhost:8000/predict \
                -H "Content-Type: application/json" \
                -d @invalid_input.json)

                if [ "\$STATUS" -eq 200 ]; then exit 1; fi
                """
            }
        }

        stage('Stop Container') {
            steps {
                sh "docker stop $CONTAINER_NAME || true"
                sh "docker rm $CONTAINER_NAME || true"
            }
        }
    }
}
