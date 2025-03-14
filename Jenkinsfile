pipeline {
    agent any
    triggers {
        pollSCM "* * * * *"
    }
    environment {
        DOCKER_USER = "aksharapuligilla"
        DOCKER_PASS = "akshara@28"
    }
    stages {
        stage('Build Application') { 
            steps {
                echo '=== Building Petclinic Application ==='
                sh 'mvn -B -DskipTests clean package' 
            }
        }
        stage('Test Application') {
            steps {
                echo '=== Testing Petclinic Application ==='
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo '=== Building Petclinic Docker Image ==='
                script {
                    app = docker.build("${DOCKER_USER}/petclinic-spinnaker-jenkins")
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo '=== Pushing Petclinic Docker Image ==='
                script {
                    GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true).trim()
                    SHORT_COMMIT = "${GIT_COMMIT_HASH[0..7]}"
                    
                    sh "echo '${DOCKER_PASS}' | docker login -u '${DOCKER_USER}' --password-stdin"
                    app.push("$SHORT_COMMIT")
                    app.push("latest")
                }
            }
        }
        stage('Remove local images') {
            steps {
                echo '=== Delete the local docker images ==='
                sh("docker rmi -f ${DOCKER_USER}/petclinic-spinnaker-jenkins:latest || :")
                sh("docker rmi -f ${DOCKER_USER}/petclinic-spinnaker-jenkins:$SHORT_COMMIT || :")
            }
        }
    }
}
