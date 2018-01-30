def REPO = ''
def PACKAGE = ''
def SOURCE_PATH = ''
def IS_MOVE = 'false'
def IS_DELETE = 'false'
def IS_BUILD = 'false'
def IS_ADD = 'false'
def MOVE_TO_REPO = ''

pipeline {
    agent any
    options {
        skipDefaultCheckout()
        timestamps()
    }
    stages {
        stage('Checkout') {
            steps {
                script {
                    checkout scm

                    def currentCommit = sh(returnStdout: true, script: 'git rev-parse @').trim()
                    echo "currentCommit: ${currentCommit}"

                    def changedFilesStatus = sh(returnStdout: true, script: "git show --pretty=format: --name-status ${currentCommit}").tokenize('\n')
                    echo "changedFilesStatus: " + changedFilesStatus

                    for ( int i = 0; i < changedFilesStatus.size(); i++ ) {
                        def entry = changedFilesStatus[i].split()
                        def status = entry[0]
                        def filePath = []
                        for ( int j = 1; j < entry.size(); j++ ) {
                            if ( entry[j].contains('PKGBUILD') ){
                                filePath << entry[j].minus('/PKGBUILD')
                            }
                        }
                        def pathSize = filePath.size()
                        if ( pathSize > 1 ){
                            if ( status.contains('R') ) {
                                IS_MOVE = 'true'
                                if ( filePath[0].contains('staging') ) {
                                    SOURCE_PATH = filePath[1]
                                    REPO = 'goblins'
                                    MOVE_TO_REPO = 'gremlins'
                                } else if ( filePath[0].contains('testing') ) {
                                    SOURCE_PATH = filePath[1]
                                    if ( SOURCE_PATH.contains('staging') ) {
                                        REPO = 'gremlins'
                                        MOVE_TO_REPO = 'goblins'
                                    } else if ( SOURCE_PATH.contains('core') ) {
                                        REPO = 'gremlins'
                                        MOVE_TO_REPO = 'system'
                                    } else if ( SOURCE_PATH.contains('extra') ) {
                                        REPO = 'gremlins'
                                        MOVE_TO_REPO = 'world'
                                    }
                                }
                            }
                        } else {
                            if ( filePath[0] != null ) {
                                SOURCE_PATH = filePath[0]
                                def buildInfo = SOURCE_PATH.tokenize('/')
                                PACKAGE = buildInfo[0]
                                def repoNode = buildInfo[1]
                                REPO = buildInfo[2]
                                if ( repoNode == 'repos' ) {
                                    
                                    if ( status == 'A' ) {
                                        IS_BUILD = 'true'
                                        if ( REPO.contains('staging') ) {
                                            REPO = 'goblins'
                                        } else if ( REPO.contains('testing') ) {
                                            REPO = 'gremlins'
                                        } else if ( REPO.contains('core') ) {
                                            REPO = 'system'
                                        } else if ( REPO.contains('extra') ) {
                                            REPO = 'world'
                                        }
                                    }
                                    
                                    if ( status == 'M' ) {
                                        if ( REPO.contains('staging') ) {
                                            IS_BUILD = 'true'
                                            REPO = 'goblins'
                                        } else if ( REPO.contains('testing') ) {
                                            IS_BUILD = 'true'
                                            REPO = 'gremlins'
                                        } else if ( REPO.contains('core') ) {
                                            IS_ADD = 'true'
                                            REPO = 'system'
                                        } else if ( REPO.contains('extra') ) {
                                            IS_BUILD = 'true'
                                            REPO = 'world'
                                        }
                                    }
                                    
                                    if ( status == 'D' ) {
                                        IS_DELETE = 'true'
                                        if ( REPO.contains('staging') ) {
                                            REPO = 'goblins'
                                        } else if ( REPO.contains('testing') ) {
                                            REPO = 'gremlins'
                                        } else if ( REPO.contains('core') ) {
                                            REPO = 'system'
                                        } else if ( REPO.contains('extra') ) {
                                            REPO = 'world'
                                        }
                                        SOURCE_PATH = "${PACKAGE}/trunk"
                                    }
                                    
                                }
                            }
                        }
                        echo "PACKAGE: ${PACKAGE}"
                        echo "SOURCE_PATH: ${SOURCE_PATH}"
                        echo "REPO: ${REPO}"
                        echo "IS_BUILD: ${IS_BUILD}"
                        echo "IS_ADD: ${IS_ADD}"
                        echo "IS_MOVE: ${IS_MOVE} MOVE_TO_REPO: ${MOVE_TO_REPO}"
                        echo "IS_DELETE: ${IS_DELETE}"
                    }
                }
            }
        }
        stage('Build') {
            environment {
                BUILDBOT_GPGP = credentials('BUILDBOT_GPGP')
            }
            when {
                expression { return  IS_BUILD == 'true' }
                expression { return  IS_ADD == 'false' }
                expression { return  IS_MOVE == 'false' }
                expression { return  IS_DELETE == 'false' }
            }
            steps {
                script {
                    dir("${SOURCE_PATH}") {
                        if ( PACKAGE != '' ) {
                            echo "buildpkg2 -r ${REPO}"
                        }
                    }
                }
            }
            post {
                success {
                    script {
                        dir("${SOURCE_PATH}") {
                            if ( PACKAGE != '' ) {
                                echo "deploypkg2 -a -d ${REPO}"
                            }
                        }
                    }
                }
            }
        }
        stage('Move') {
            when {
                expression { return  IS_BUILD == 'false' }
                expression { return  IS_ADD == 'false' }
                expression { return  IS_MOVE == 'true' }
                expression { return  IS_DELETE == 'false' }
            }
            steps {
                script {
                    dir("${SOURCE_PATH}") {
                        echo "deploypkg2 -m -s ${REPO} -d ${MOVE_TO_REPO}"
                    }
                }
            }
        }
        stage('Add') {
            when {
                expression { return  IS_BUILD == 'false' }
                expression { return  IS_ADD == 'true' }
                expression { return  IS_MOVE == 'false' }
                expression { return  IS_DELETE == 'false' }
            }
            steps {
                script {
                    dir("${SOURCE_PATH}") {
                        echo "deploypkg2 -a -d ${REPO}"
                    }
                }
            }
        }
        stage('Remove') {
            when {
                expression { return  IS_BUILD == 'false' }
                expression { return  IS_ADD == 'false' }
                expression { return  IS_MOVE == 'false' }
                expression { return  IS_DELETE == 'true' }
            }
            steps {
                script {
                    dir("${SOURCE_PATH}") {
                        echo "deploypkg2 -r -d ${REPO}"
                    }
                }
            }
        }
    }
}
