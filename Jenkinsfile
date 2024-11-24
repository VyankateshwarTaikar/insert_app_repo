@Library('my-shared-library') _

pipeline {

    agent any

    parameters {
        choice(
            name: 'action',
            choices: 'create\ndelete',
            description: 'Choose create/Destroy'
        )
        string(
            name: 'ImageName',
            description: "Name of the docker build",
            defaultValue: 'javapp'
        )
        string(
            name: 'ImageTag',
            description: "Tag of the docker build",
            defaultValue: 'v1'
        )
        string(
            name: 'DockerHubUser',
            description: "Name of the Application",
            defaultValue: 'YourNameofApp'
        )
    }

    stages {

        stage('Git Checkout') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                gitCheckout(
                    branch: "main",
                    url: "https://github.com/VyankateshwarTaikar/insert_app_repo.git"
                )
            }
        }

        stage('Unit Test Maven') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    mvnTest()
                }
            }
        }

        stage('Integration Test Maven') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    mvnIntegrationTest()
                }
            }
        }

        stage('Static Code Analysis: Sonarqube') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    def SonarQubecredentialsId = 'sonarqube-api'
                    statiCodeAnalysis(SonarQubecredentialsId)
                }
            }
        }

        stage('Quality Gate Status Check: Sonarqube') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    def SonarQubecredentialsId = 'sonarqube-api'
                    QualityGateStatus(SonarQubecredentialsId)
                }
            }
        }

        stage('Maven Build: Maven') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    mvnBuild()
                }
            }
        }

        stage('Docker Image Build') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    dockerBuild("${params.ImageName}", "${params.ImageTag}", "${params.DockerHubUser}")
                }
            }
        }

        stage('Docker Image Scan: Trivy') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    dockerImageScan("${params.ImageName}", "${params.ImageTag}", "${params.DockerHubUser}")
                }
            }
        }

        stage('Pushing JFrog File') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    sh 'curl -X PUT -u admin:password -T /var/lib/jenkins/workspace/demo_3.0/target/kubernetes-configmap-reload-0.0.1-SNAPSHOT.jar "http://54.163.62.92:8082/artifactory/example-repo-local/kubernetes-configmap-reload-0.0.1-SNAPSHOT.jar"'
                }
            }
        }

        stage('Docker Image Push: DockerHub') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    dockerImagePush("${params.ImageName}", "${params.ImageTag}", "${params.DockerHubUser}")
                }
            }
        }

        stage('Docker Image Cleanup: DockerHub') {
            when {
                expression { params.action == 'create' }
            }
            steps {
                script {
                    dockerImageCleanup("${params.ImageName}", "${params.ImageTag}", "${params.DockerHubUser}")
                }
            }
        }
    }
}
