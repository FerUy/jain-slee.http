pipeline {
    agent any
    tools {
        jdk 'JDK 11'
        maven 'Maven_3.8.5'
    }
    parameters {
        string(name: 'version', defaultValue: '7.2.0', description: 'The major version number for the JSLEE HTTP')
    }
    options {
        timeout(time: 1, unit: 'HOURS')
       	buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '30', numToKeepStr: '10'))
    }
    stages {
        stage('Set Version') {
            steps{
                script{
                    currentBuild.displayName = "${params.version}-${BUILD_NUMBER}"
                    currentBuild.description = "JAIN SLEE HTTP (${env.BRANCH_NAME})"
                }
                sh "mvn versions:set -DnewVersion=${params.version}-${BUILD_NUMBER}"
            }
        }
        stage('Build') {
            steps{
                sh "mvn clean install -DskipTests"
                sh "mvn versions:commit"
            }
        }
        stage('Release'){
            steps{
                echo "Building a release version of #${params.version}-${BUILD_NUMBER}"
                sh "mvn clean install -Prelease -Drelease.dir=../../../${params.version}-${BUILD_NUMBER}"
   				echo "Building a release version completed."
            }
        }
        stage('Zip artifact'){
            steps{
                script {
                    VERSION_FOLDER = "${params.version}-${BUILD_NUMBER}"
                }
                sh " find ${VERSION_FOLDER}/resources -maxdepth 1 -type d -exec zip -r -qq {}.zip {} \\;"
                sh " find ${VERSION_FOLDER}/extra -maxdepth 1 -type d -exec zip -r -qq {}.zip {} \\;"
                sh " find ${VERSION_FOLDER}/enablers -maxdepth 1 -type d -exec zip -r -qq {}.zip {} \\;"
                // sh " find ${VERSION_FOLDER} -iname \"*.zip\" -exec mv {} ${VERSION_FOLDER} \\;"
                //sh " find ${VERSION_FOLDER} -maxdepth 1 -type d -exec rm -rf {} \\;"
            }
        }
        stage('Save artifact'){
            steps{
                archiveArtifacts artifacts: "${params.version}-${BUILD_NUMBER}/**/*.zip", followSymlinks: false, onlyIfSuccessful: true
            }
        }

        stage('Push to Repo') {
            when {anyOf{branch 'master'; branch 'release'}}
            steps {
                script {
                    REMOTEPATH = "/var/www/html/NAIKERI/jain_slee_http/${params.version}-${BUILD_NUMBER}"
                }

                sh "mkdir -p /var/www/html/NAIKERI/jain_slee_http/${params.version}-${BUILD_NUMBER}"
                sh "cp ${params.version}-${BUILD_NUMBER}/**/*.zip /var/www/html/NAIKERI/jain_slee_http/${params.version}-${BUILD_NUMBER}"
                /*sshagent (credentials: ['4e708f2a-8b37-414f-a6d4-787690b87738']) {
                    sh 'ssh -o StrictHostKeyChecking=no -l fer 127.0.0.1 uname -a'
                	sh "scp release/Naikeri-jSS7-${params.jSS7_MAJOR_VERSION_NUMBER}-${BUILD_NUMBER}.zip fer@127.0.0.1:/var/www/html/NAIKERI/jain_slee_http/${params.version}-${BUILD_NUMBER/"
                }*/
                sh "rm -rf ${params.version}-${BUILD_NUMBER}"

            }
        }
    }
    post {
        always {
            sh "rm -rf ${params.version}-${BUILD_NUMBER}"
        }
    }
}