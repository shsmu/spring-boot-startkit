#!groovy​

properties([[$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', numToKeepStr: '10']]])

stage('build') {
    node {
        checkout scm
        def v = version()
        currentBuild.displayName = "${env.BRANCH_NAME}-${v}-${env.BUILD_NUMBER}"
        mvn "clean "
    }
}

stage('build docker image') {
    node {
        mvn "clean package docker:build -DskipTests"
    }
}

def branch_type = get_branch_type "${env.BRANCH_NAME}"
def branch_deployment_environment = get_branch_deployment_environment branch_type

if (branch_deployment_environment) {
    stage('deploy') {
        if (branch_deployment_environment == "prod") {
            timeout(time: 1, unit: 'DAYS') {
                input "Deploy to ${branch_deployment_environment} ?"
            }
        }
        node {
            sh "echo Deploying to ${branch_deployment_environment}"
            //TODO specify the deployment
        }
    }

    if (branch_deployment_environment != "prod") {
        stage('integration tests') {
            node {
                sh "echo Running integration tests in ${branch_deployment_environment}"
                //TODO do the actual tests
            }
        }
    }
}

if (branch_type == "dev") {
    stage('start release') {
        timeout(time: 1, unit: 'HOURS') {
            input "Do you want to start a release?"
        }
        node {
                sh "echo  Deploying to ${branch_deployment_environment}"
        }
    }
    stage('Push docker images') {
        node {
                sh "echo  Deploying to ${branch_deployment_environment}"
              	withCredentials([string(credentialsId: 'dockerHubPwd', variable: 'dockerHubPwd')]) {
                      sh "docker login -u shsmu -p ${dockerHubPwd}"
                }
              	sh "docker tag shsmu/springbootstartkit:latest  shsmu/springbootstartkit:${currentBuild.displayName}; docker push shsmu/springbootstartkit:${currentBuild.displayName}"
        }
    }
    stage('deploy') {
        node {
                sh "echo  Deploying to ${branch_deployment_environment}"
                def dockerRun = 'docker run -d -p 18080:8080 --name spring-boot-startkit shsmu/springbootstartkit '
                sshagent(['dev.sanyu.com']) {
                    sh "ssh -o StrictHostKeyChecking=no root@dev.sanyu.com ${dockerRun}"
                }
        }
    }
}

if (branch_type == "release") {
    stage('finish release') {
        timeout(time: 1, unit: 'HOURS') {
            input "Is the release finished?"
        }
        node {
                sh "echo  Deploying to ${branch_deployment_environment}"
                //TODO do the actual tests

        }
    }
}

if (branch_type == "hotfix") {
    stage('finish hotfix') {
        timeout(time: 1, unit: 'HOURS') {
            input "Is the hotfix finished?"
        }
        node {
                sh "echo  Deploying to ${branch_deployment_environment}"
                //TODO do the actual tests

        }
    }
}

// Utility functions
def get_branch_type(String branch_name) {
    //Must be specified according to <flowInitContext> configuration of jgitflow-maven-plugin in pom.xml
    def dev_pattern = ".*develop"
    def release_pattern = ".*release/.*"
    def feature_pattern = ".*feature/.*"
    def hotfix_pattern = ".*hotfix/.*"
    def master_pattern = ".*master"
    if (branch_name =~ dev_pattern) {
        return "dev"
    } else if (branch_name =~ release_pattern) {
        return "release"
    } else if (branch_name =~ master_pattern) {
        return "master"
    } else if (branch_name =~ feature_pattern) {
        return "feature"
    } else if (branch_name =~ hotfix_pattern) {
        return "hotfix"
    } else {
        return null;
    }
}

def get_branch_deployment_environment(String branch_type) {
    if (branch_type == "dev") {
        return "dev"
    } else if (branch_type == "release") {
        return "staging"
    } else if (branch_type == "master") {
        return "prod"
    } else {
        return null;
    }
}

def mvn(String goals) {
    def mvnHome = tool "Maven"
    def javaHome = tool "JDK"

    withEnv(["JAVA_HOME=${javaHome}", "PATH+MAVEN=${mvnHome}/bin"]) {
        sh "mvn -B ${goals}"
    }
}

def version() {
    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
    return matcher ? matcher[0][1] : null
}