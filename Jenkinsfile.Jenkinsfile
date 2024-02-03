pipeline {
    agent {
        dockerfile {
          filename "Dockerfile"
          args "-u root" // root is necessary for virtualenv
        }
    }
    environment {
        SPHINX_DIR  = '.'
        BUILD_DIR   = './_built'
        SOURCE_DIR  = './source'
        // DEPLOY_HOST = 'deployer@www.example.com:/path/to/docs/'
        // deploy to local directory for testing
        DEPLOY_HOST = '/tmp/sj_docs/'
    }
    stages {
        stage('Install Dependencies') {
            steps {
                sh '''
                   virtualenv pyenv
                   . pyenv/bin/activate
                   pip install -r ${SPHINX_DIR}/requirements.txt
                '''
            }
        }
        stage('Build') {
            steps {
                // clear out old files
                sh 'rm -rf ${BUILD_DIR}'
                sh 'rm -f ${SPHINX_DIR}/sphinx-build.log'

                sh '''
                   ${WORKSPACE}/pyenv/bin/sphinx-build \
                   -q -w ${SPHINX_DIR}/sphinx-build.log \
                   -b html \
                   -d ${BUILD_DIR}/doctrees ${SOURCE_DIR} ${BUILD_DIR}
                '''
            }
            post {
                failure {
                    sh 'cat ${SPHINX_DIR}/sphinx-build.log'
                }
            }
        }
        stage('Deploy-Local') {
            steps {
                sh '''#!/bin/bash
                   rm -f ${SPHINX_DIR}/rsync.log
                   cp -r ${BUILD_DIR}/ ${DEPLOY_HOST}
                '''
            }
            post {
                always {
                    sh 'cat ${SPHINX_DIR}/rsync.log'
                }
                failure {
                    sh 'cat ${SPHINX_DIR}/rsync.log'
                }
            }
//         stage('Deploy') {
//             steps {
//                 sshagent(credentials: ['deployer']) {
//                    sh '''#!/bin/bash
//                       rm -f ${SPHINX_DIR}/rsync.log
//                       RSYNCOPT=(-aze 'ssh -o StrictHostKeyChecking=no')
//                       rsync "${RSYNCOPT[@]}" \
//                       --exclude-from=${SPHINX_DIR}/rsync-exclude.txt \
//                       --log-file=${SPHINX_DIR}/rsync.log \
//                       --delete \
//                       ${BUILD_DIR}/ ${DEPLOY_HOST}
//                     '''
//                 }
//             }
//             post {
//                 failure {
//                     sh 'cat ${SPHINX_DIR}/rsync.log'
//                 }
//             }
//         }
    }
}