/*
 * Normal Jenkinsfile that will build and do Policy and SCA scans
 */

pipeline {
    agent any

    environment {
        VERACODE_APP_NAME = 'DVWA'      // App Name in the Veracode Platform
    }

    // this is optional on Linux, if jenkins does not have access to your locally installed docker
    //tools {
        // these match up with 'Manage Jenkins -> Global Tool Config'
        //'org.jenkinsci.plugins.docker.commons.tools.DockerTool' 'docker-latest' 
    //}

    options {
        // only keep the last x build logs and artifacts (for space saving)
        buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '20'))
    }

    stages{
        stage ('environment verify') {
            steps {
                script {
                    if (isUnix() == true) {
                        sh 'pwd'
                        sh 'ls -la'
                        sh 'echo $PATH'
                    }
                    else {
                        bat 'dir'
                        bat 'echo %PATH%'
                    }
                }
            }
        }

        stage ('Veracode scan') {
            steps {
                // zip archive for Veracode scanning.  Only include stuff we need,
                //  aka skip things like the external directory
                zip zipFile: 'upload.zip', archive: false, glob: 'app/*.php,app/dvwa/**,app/hackable/**,app/vulnerabilities/**'

                script {
                    if(isUnix() == true) {
                        env.HOST_OS = 'Unix'
                    }
                    else {
                        env.HOST_OS = 'Windows'
                    }
                }

                echo 'Veracode scanning'
                withCredentials([ usernamePassword ( 
                    credentialsId: 'veracode_login', usernameVariable: 'VERACODE_API_ID', passwordVariable: 'VERACODE_API_KEY') ]) {
                        // fire-and-forget 
                        veracode applicationName: "${VERACODE_APP_NAME}", criticality: 'VeryHigh', debug: true, fileNamePattern: '', pHost: '', pPassword: '', pUser: '', replacementPattern: '', sandboxName: '', scanExcludesPattern: '', scanIncludesPattern: '', scanName: "${BUILD_TAG}-${env.HOST_OS}", uploadExcludesPattern: '', uploadIncludesPattern: 'upload.zip', useIDkey: true, vid: "${VERACODE_API_ID}", vkey: "${VERACODE_API_KEY}"

                        // wait for scan to complete (timeout: x)
                        //veracode applicationName: "${VERACODE_APP_NAME}", criticality: 'VeryHigh', debug: true, timeout: 20, fileNamePattern: '', pHost: '', pPassword: '', pUser: '', replacementPattern: '', sandboxName: '', scanExcludesPattern: '', scanIncludesPattern: '', scanName: "${BUILD_TAG}", uploadExcludesPattern: '', uploadIncludesPattern: 'upload.zip', useIDkey: true, vid: "${VERACODE_API_ID}", vkey: "${VERACODE_API_KEY}"
                    }      
            }
        }

        // No PHP open source, so no SCA scanning

        // only works on *nix, as we're building a Linux image
        //  uses the natively installed docker
        stage ('Deploy') {
            when { expression { return (isUnix() == true) } }
            steps {
                echo 'building Docker image'
                sh 'docker version'

                ansiColor('xterm') {
                    sh 'docker build -t dvwa:${BUILD_TAG} .'
                }
                
                // split into separate stage??
                echo 'Deploying ...'
        
            }
        }
    }
}
