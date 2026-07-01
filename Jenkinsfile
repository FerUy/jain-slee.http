pipeline {
    agent any
    tools {
        jdk 'jdk-11'
        maven 'maven-3.9.12'
    }
    options {
        buildDiscarder(logRotator(daysToKeepStr: '10', numToKeepStr: '10'))
    }
    parameters {
        string(name: 'JSLEE_HTTP_VERSION', defaultValue: '7.2.0', description: 'The major version for JAIN SLEE HTTP')
    }
    environment {
        HTTP_BUILD_VERSION = "${params.JSLEE_HTTP_VERSION}-${BUILD_NUMBER}"
    }
    stages {
        stage("Build") {
            steps {
                script {
                    HTTP_BUILD_VERSION = "${params.JSLEE_HTTP_VERSION}-${BUILD_NUMBER}"
                    currentBuild.displayName = "#${HTTP_BUILD_VERSION}"
                    currentBuild.description = "JAIN SLEE HTTP (${env.BRANCH_NAME})"
                }
                sh "mvn clean install -Dmaven.test.skip=true"
            }
        }
        stage('Set Version') {
            steps {
                sh "mvn versions:set -DgenerateBackupPoms=false -DnewVersion=${HTTP_BUILD_VERSION}"
                sh "mvn versions:commit"
            }
        }
        stage("Release") {
            steps {
                sh "mvn clean install -Prelease -Drelease.dir=../../../${HTTP_BUILD_VERSION} -Dmaven.test.skip=true"
            }
        }
        stage('Zip Resources') {
            steps {
                dir("${HTTP_BUILD_VERSION}/resources") {
                    sh 'find . -maxdepth 1 -type d ! -name . -exec zip -r -qq {}.zip {} \\;'
                    sh 'find . -maxdepth 1 -type d ! -name . -exec rm -rf {} +'
                }
            }
        }
        stage('Save Artifacts') {
            steps {
                archiveArtifacts artifacts: "${HTTP_BUILD_VERSION}/", followSymlinks: false, onlyIfSuccessful: true
            }
        }
        stage('Push to Repo') {
            when{anyOf {branch 'master'; branch 'release'}}
            steps {
                sh "mkdir -p /var/www/html/NAIKERI/jain-slee.http/${HTTP_BUILD_VERSION}/"
                sh "cp -r ${HTTP_BUILD_VERSION}/ /var/www/html/NAIKERI/jain-slee.http/${HTTP_BUILD_VERSION}/"
                sh "rm -rf ${HTTP_BUILD_VERSION}"
            }
        }
    }
    post {
        success { echo "JAIN-SLEE HTTP successfully built" }
        failure { echo "Building JAIN-SLEE HTTP failed" }
    }
}
