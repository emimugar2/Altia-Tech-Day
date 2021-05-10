pipeline {
    agent {
        node {
            label 'slave-1'
            
        }
    }
    environment {
        OUTPUT = "/home/jenkins/output_NEW_web_${env.JOB_NAME}_${env.BUILD_NUMBER}"
        ENVCREDENTIAL="null"
        BRANCH = "${env.BRANCH_NAME}"
        BRANCHZIP = "web_${env.BRANCH_NAME}"
    }
    stages {
        stage('Select Env') {
            steps {
                script{
                    if (env.BRANCH_NAME == 'master')
                    {
                        ENVCREDENTIAL="web-live"
                    }
                    else if (env.BRANCH_NAME == 'develop')
                    {
                        ENVCREDENTIAL="web-dev"
                    }
                    else
                    {
                        println("Nothing to do in this branch")
                        sh "exit 1"
                    }
                }
            }
        }
        stage('Build') {
            steps {
                sh "meteor add nitrolabs:cdn"
                sh "meteor npm install"
                sh "meteor npm install --save @babel/runtime"
                sh "meteor npm install --save simpl-schema"
                sh "meteor build $OUTPUT"
                sh "pushd $OUTPUT && tar xvzf *.tar.gz && popd"
            }
        }
        stage('Setting variables') {
            steps {
                withCredentials([
                                file(credentialsId: ENVCREDENTIAL, variable: 'ebenvconfig')
                                ])
                                {
                                    sh "cp -af \$ebenvconfig $WORKSPACE"
                                }
                sh "chmod +x prepare_files.sh; bash prepare_files.sh"
                sh "cp -rpaf ${env.WORKSPACE}/.ebextensions $OUTPUT/bundle/"
                sh "cp -rpaf ${env.WORKSPACE}/.elasticbeanstalk $OUTPUT/bundle/"
                sh "cp -af ${env.WORKSPACE}/package.json $OUTPUT/bundle/"
                sh "cp -af ${env.WORKSPACE}/.npmrc $OUTPUT/bundle/"
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'ejemplo-iam-pro']]) {
                    getNameofApp()
                    sh "cd $OUTPUT/bundle/; eb deploy $nameOfApp"
                }
            }
        }
    }
    post {
        always{
            cleanWS()
        }

    }
}

def cleanWS(){
  stage ('Clean Workspace'){
    sh("#!/bin/sh -e\n rm -fR ${env.WORKSPACE}/* && rm -fR ${OUTPUT}")
  }
}

def getNameofApp(){
    nameOfApp = sh ( returnStdout: true, script : "cat .elasticbeanstalk/config.yml | grep environment | cut -d ':' -f2 | sed 's/ //g' | tr -d '\r'")
}
