pipeline {
	agent any
	environment {
		DOCKERHUB_USER = "partico"
		BUILD_HOST = "root@192.168.64.4"
		PROD_HOST = "root@192.168.64.5"
	}
	stages {
		stage('Initialize') {
			steps {
				script {
					BUILD_TIMESTAMP = sh(script: "date +%Y%m%d-%H%M%S", returnStdout: true).trim()
				}
			}
		}
		stage('Pre check') {
			steps {
				sh "test -f ~/.docker/config.json"
				sh "cat ~/.docker/config.json | grep docker.io"
			}
		}
		stage('Build') {
			steps {
				sh "cat docker-compose.build.yml"
				withEnv(["DOCKER_HOST=ssh://${BUILD_HOST}"]) {
					sh "docker compose -f docker-compose.build.yml down"
					sh "docker volume prune -f"
					sh "docker compose -f docker-compose.build.yml build"
					sh "docker compose -f docker-compose.build.yml up -d"
					sh "docker compose -f docker-compose.build.yml ps"
				}
			}
		}
		stage('Test') {
			steps {
				withEnv(["DOCKER_HOST=ssh://${BUILD_HOST}"]) {
					sh "sleep 10"
					sh "docker compose -f docker-compose.build.yml exec -T apptest pytest -v test_app.py"
					sh "docker compose -f docker-compose.build.yml exec -T webtest pytest -v test_static.py"
					sh "docker compose -f docker-compose.build.yml exec -T webtest pytest -v test_selenium.py"
					sh "docker compose -f docker-compose.build.yml down"
				}	
			}
		}
		stage('Register') {
			steps {
				sh "docker -H ssh://${BUILD_HOST} tag dockerkvs_web ${DOCKERHUB_USER}/dockerkvs_web:${BUILD_TIMESTAMP}"
				sh "docker -H ssh://${BUILD_HOST} tag dockerkvs_app ${DOCKERHUB_USER}/dockerkvs_app:${BUILD_TIMESTAMP}"
				sh "docker -H ssh://${BUILD_HOST} push ${DOCKERHUB_USER}/dockerkvs_web:${BUILD_TIMESTAMP}"
				sh "docker -H ssh://${BUILD_HOST} push ${DOCKERHUB_USER}/dockerkvs_app:${BUILD_TIMESTAMP}"
			}
		}
		stage('Deploy') {
			steps {
				sh "cat docker-compose.prod.yml"
				sh "echo 'DOCKERHUB_USER=${DOCKERHUB_USER}' > .env"
				sh "echo 'BUILD_TIMESTAMP=${BUILD_TIMESTAMP}' >> .env"
				sh "cat .env"
				sh "docker compose -H ssh://${PROD_HOST} -f docker-compose.prod.yml up -d"
				sh "docker compose -H ssh://${PROD_HOST} -f docker-compose.prod.yml ps"
			}
		}
	}
}
