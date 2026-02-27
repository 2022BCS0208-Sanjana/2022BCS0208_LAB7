pipeline {
    agent any

    environment {
        IMAGE_NAME = "2022bcs0208sanjana/wine_predict_2022bcs0208:latest"
        CONTAINER_NAME = "Lab7_mlops"
        API_URL = "http://localhost:8000"
    }

    stages {

        stage('Pull Image') {
            steps {
                sh 'docker pull $IMAGE_NAME'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                echo "Removing old container if exists..."
                docker rm -f $CONTAINER_NAME || true

                echo "Starting container..."
                docker run -d --name $CONTAINER_NAME -p 8000:8000 $IMAGE_NAME
                '''
            }
        }

        stage('Wait for API') {
            steps {
                sh '''
                echo "Waiting for API to be ready..."
                sleep 5

                for i in {1..15}; do
                    if curl -s -X POST $API_URL/predict \
                        -H "Content-Type: application/json" \
                        -d @valid_input.json > /dev/null; then
                        echo "API is ready"
                        exit 0
                    fi
                    echo "Retry $i..."
                    sleep 3
                done

                echo "API did not start in time"
                exit 1
                '''
            }
        }

        stage('Valid Inference Test') {
            steps {
                sh '''
                echo "Sending valid inference request..."
                RESPONSE=$(curl -s -X POST $API_URL/predict \
                    -H "Content-Type: application/json" \
                    -d @valid_input.json)

                echo "Response: $RESPONSE"

                echo "$RESPONSE" | grep -q "prediction"
                '''
            }
        }

        stage('Invalid Inference Test') {
            steps {
                sh '''
                echo "Sending invalid inference request..."
                STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
                    -X POST $API_URL/predict \
                    -H "Content-Type: application/json" \
                    -d @invalid_input.json)

                echo "HTTP Status: $STATUS"

                if [ "$STATUS" -eq 200 ]; then
                    echo "Invalid input incorrectly accepted"
                    exit 1
                fi
                '''
            }
        }
    }

    post {
        always {
            sh '''
            echo "Cleaning up container..."
            docker rm -f $CONTAINER_NAME || true
            '''
        }
        success {
            echo "✅ Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}
