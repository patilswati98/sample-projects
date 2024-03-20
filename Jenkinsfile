pipeline {
    agent any
    parameters {
        string(name: 'GIT_REPO', defaultValue: 'https://github.com/optit-cloud-team/sample-projects.git', description: 'Git Repository URL')
        string(name: 'GIT_BRANCH', defaultValue: 'main', description: 'Branch to build')
    }
    options {
        timeout(time: 1, unit: 'HOURS')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
        githubProjectProperty(
            displayName: '',
            projectUrlStr: '${params.GIT_REPO}'
        )
    }
    def job_name = "${env.JOB_NAME.split('/')[-1]}"
    def commit_hash = "${env.GIT_COMMIT.substring(0, 7)}"
    def build_number = "${env.BUILD_NUMBER}"
    def docker_image = "${job_name}:${commit_hash}-${build_number}"
    stages {
        stage('Checkout') {
            steps {
                script {
                    git (url: "${params.GIT_REPO}", branch: "${params.GIT_BRANCH}")
                }
            }
        }
        stage('Build') {
            steps {
                script {
                    
                    sh 'gradle clean build'
                    artifact 'build/libs/*.jar'
                    
                }
            }
        }
        stage('Create Docker Image') {
            steps {
                script {
                    if (!fileExists('Dockerfile')) {
                        
                        writeFile(file: 'Dockerfile', text: '''
                        FROM openjdk:17-jdk-slim
                        WORKDIR /app
                        COPY target/*.jar app.jar
                        ENTRYPOINT ["java", "-jar", "app.jar"]
                        ''')
                        
                    else {
                        print('Dockerfile already exists')
                    }
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    sh 'docker build -t ${docker_image} .'
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    sh 'docker push ${docker_image}'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    sh 'kubectl create deployment $job_name --image=${docker_image}'
                    sh 'kubectl expose deployment $job_name --type=ClusterIP --port=8080'
                }
            }
        }
        
    }
}
