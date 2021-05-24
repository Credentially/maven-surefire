#!groovy

env.DOCKER_HOST_IP = '172.17.0.1'

node('jenkins-build-agent') {
    // This limits build concurrency to 1 per branch
    properties([disableConcurrentBuilds()])

    env.JAVA_HOME = "${tool 'jdk8'}"
    env.PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
    env.jdkHome="$JAVA_HOME"
    // TODO: permission denied is thrown using tools
    //env.GIT_HOME="${tool 'git'}"
    //env.PATH="${env.GIT_HOME}/bin:${env.PATH}"

    timeout(55 /* minutes */) {
        // Need to replace the '%2F' used by Jenkins to deal with / in the path (e.g. story/...)
        // because tests that do getResource will escape the % again, and the test files can't be found.
        // See https://issues.jenkins-ci.org/browse/JENKINS-34564 for more.
        ws("workspace/${env.JOB_NAME}") {
            stage('Check Environment') {
                sh 'java -version'
                sh 'git --version'
                sh 'whoami'
                sh 'env'
            }

            stage('Checkout') {
                git branch: getBranchName(),
                    credentialsId: 'github-token',
                    url: 'https://github.com/Credentially/maven-surefire.git'
            }


            stage('Build') {
                mvn ' -T 2C site site:stage -Dmaven.test.skip=true -Drat.skip=true -P reporting,run-its'
            }

            stage('Deploy') {
                mvn '-Dmaven.test.skip=true -Dmaven.install.skip=true deploy'
            }

        }
    }
}

def mvn(args) {
      //github-token belong to Andrey Beletsky user. Maven Github packages does not accept Github App token github-authentication currently.
        withCredentials([usernamePassword(credentialsId: 'github-token', passwordVariable: 'pswd', usernameVariable: 'usrn')]) {
          withCredentials([file(credentialsId: 'settings_xml', variable: 'MAVEN_SETTINGS_XML')]) {
              withEnv(['MAVEN_OPTS=-Xmx2g', "PATH+MAVEN=${tool 'maven-3.6.3'}/bin"]) {
                sh 'echo "MAVEN_OPTS=$MAVEN_OPTS"'
                sh 'echo "PATH=$PATH"'

                sh "mvn -U -gs $MAVEN_SETTINGS_XML $args"
              }
          }
      }
}

/** It is used to hide sh command if it has secret data */
def runSecretScript(String label, String scriptBody) {
    sh label: label, script: "#!/bin/sh -e \n $scriptBody"
}

def getBranchName() {
    env.BRANCH_NAME.startsWith('PR-') ? env.CHANGE_BRANCH : env.BRANCH_NAME
}


