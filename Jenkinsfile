@Library("shared-library") _
pipeline {
    agent any
    
    environment{
        SONAR_HOME = tool "SonarQube"
    }
    
    parameters {
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: '1.1', description: 'Setting docker image for latest push')
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: '1.1', description: 'Setting docker image for latest push')
    }
    
    stages {
        stage("Validate Parameters") {
            steps {
                script {
                    if (params.FRONTEND_DOCKER_TAG == '' || params.BACKEND_DOCKER_TAG == '') {
                        error("FRONTEND_DOCKER_TAG and BACKEND_DOCKER_TAG must be provided.")
                    }
                }
            }
        }
        stage("Workspace cleanup"){
            steps{
                script{
                    cleanWs()
                }
            }
        }
        
       stage('Git: Code Checkout') {
            steps {
                script{
                    git_clone("https://github.com/coooolk/Wanderlust-Mega-Project.git","main")
                }
            }
        } 
        
        stage("Trivy: Filesystem scan"){
            steps{
                script{
                    trivy_scan("fs", ".")
                }
            }
        }

       /* stage('Install npm Dependencies') {
            steps {
                script {
                    sh 'npm install'
                }
            }
       } */

        stage("OWASP: Dependency check"){
            steps{
                script{
                    owasp_dependency()
                }
            }
        }
        
        stage("SonarQube: Code Analysis"){
            steps{
                script{
                    sonarqube_analysis("SonarQube","wanderlust","wanderlust")
                }
            }
        }
        
        stage("SonarQube: Code Quality Gates"){
            steps{
                script{
                    sonarqube_code_quality()
                }
            }
        }
        
        stage('Exporting environment variables') {
            parallel{
                stage("Backend env setup"){
                    steps {
                        script{
                            dir("Automations"){
                                sh "bash updatebackendnew.sh"
                            }
                        }
                    }
                }
                
                stage("Frontend env setup"){
                    steps {
                        script{
                            dir("Automations"){
                                sh "bash updatefrontendnew.sh"
                            }
                        }
                    }
                }
            }
        }
        
        stage("Docker: Build Images"){
            steps{
                script{
                        dir('backend'){
                            docker_build("coooolkpk", "wanderlust-backend-beta", "${params.BACKEND_DOCKER_TAG}")
                        }
                    
                        dir('frontend'){
                            docker_build("coooolkpk", "wanderlust-frontend-beta", "${params.FRONTEND_DOCKER_TAG}")
                        }
                }
            }
        }
        
        stage("Docker: Push to DockerHub"){
            steps{
                script{
                    docker_push("coooolkpk", "wanderlust-backend-beta", "${params.BACKEND_DOCKER_TAG}") 
                    docker_push("coooolkpk", "wanderlust-frontend-beta", "${params.FRONTEND_DOCKER_TAG}")
                }
            }
        }
    }
    post{
        success{
            archiveArtifacts artifacts: '*.xml', followSymlinks: false
            build job: "Wanderlust-CD", parameters: [
                string(name: 'FRONTEND_DOCKER_TAG', value: "${params.FRONTEND_DOCKER_TAG}"),
                string(name: 'BACKEND_DOCKER_TAG', value: "${params.BACKEND_DOCKER_TAG}")
            ]
        }
    }
}
