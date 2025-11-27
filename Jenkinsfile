pipeline {
    agent any

    tools {
        maven 'M2_HOME'
    }

    environment {
        MAVEN_HOME = "${tool 'M2_HOME'}"
        PATH = "${env.MAVEN_HOME}/bin:${env.PATH}"

        DOCKER_IMAGE = "mayamarzouki/student-management"
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Maya-Marzouki/DevOpsPipeline-.git'
            }
        }

        stage('Package App') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Prepare Docker Context') {
            steps {
                sh """
                    mkdir -p docker-build
                    cp target/*SNAPSHOT.jar docker-build/app.jar

                    cat > docker-build/Dockerfile << EOF
FROM eclipse-temurin:17-jre-alpine
COPY app.jar app.jar
EXPOSE 8089
ENTRYPOINT ["java", "-jar", "app.jar"]
EOF
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    echo "🐳 Building Docker image..."
                    docker build -t ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} docker-build
                    docker tag ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} ${env.DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                sh """
                    echo "🔐 DockerHub Login..."
                    echo "${env.DOCKERHUB_PASSWORD}" | docker login -u ${env.DOCKERHUB_USERNAME} --password-stdin

                    echo "🚀 Pushing images..."
                    docker push ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}
                    docker push ${env.DOCKER_IMAGE}:latest
                """
            }
        }

    }

    post {
        success {
            echo """
            🎉 Docker Image Built & Pushed Successfully!

            Image pushed:
            - ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}
            - ${env.DOCKER_IMAGE}:latest
            """
        }
        failure {
            echo "❌ Pipeline failed."
        }
    }
}
