def ADD_REPO = ''
def RM_REPO = ''
def IS_ADD = 'false'
def IS_REMOVE = 'false'
def IS_BUILD = 'false'
def PKG_TRUNK = ''
def PKG_PATH = ''

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
                    def changedPkgStatus = []
                    def pkgPath = []
                    int entryCount = 0
                    for ( int i = 0; i < changedFilesStatus.size(); i++ ) {
                        def entry = changedFilesStatus[i].split()
                        def fileStatus = entry[0]
                        entryCount = entry.size()
                        for ( int j = 1; j < entry.size(); j++ ) {
                            if ( entry[j].contains('/PKGBUILD') && entry[j].contains('/repos') ){
                                changedPkgStatus << "${fileStatus} " + entry[j].minus('/PKGBUILD')
                                pkgPath << entry[j].minus('/PKGBUILD')
                            }
                        }
                    }

                    int pkgCount = changedPkgStatus.size()
                    int pkgPathCount = pkgPath.size()
                    echo "pkgCount: ${pkgCount}"
                    echo "entryCount: ${entryCount}"
                    echo "pkgPathCount: ${pkgPathCount}"
                    echo "changedPkgStatus: ${changedPkgStatus}"

                    if ( pkgCount > 0 ) {

                        if ( entryCount == 2 && pkgCount == 2 ) {
                            def pkgEntry1 = changedPkgStatus[0].split()
                            def pkgEntry2 = changedPkgStatus[1].split()
                            def srcPath = []
                            def pkgStatus = []
                            srcPath << pkgEntry1[1]
                            srcPath << pkgEntry2[1]
                            pkgStatus << pkgEntry1[0]
                            pkgStatus << pkgEntry2[0]
                            def buildInfo1 = srcPath[0].tokenize('/')
                            def buildInfo2 = srcPath[1].tokenize('/')
                            def repoName1 = buildInfo1[2]
                            def repoName2 = buildInfo2[2]

                            if ( pkgStatus[0] == 'M' ) {
                                IS_ADD = 'true'
                                if ( repoName1.contains('staging') ) {
                                    ADD_REPO = 'goblins'
                                } else if ( repoName1.contains('testing') ) {
                                    ADD_REPO = 'gremlins'
                                } else if ( repoName1.contains('core') ) {
                                    ADD_REPO = 'system'
                                } else if ( repoName1.contains('extra') ) {
                                    ADD_REPO = 'world'
                                }
                            } else if ( pkgStatus[1] == 'M' ) {
                                IS_ADD = 'true'
                                if ( repoName2.contains('staging') ) {
                                    ADD_REPO = 'goblins'
                                } else if ( repoName2.contains('testing') ) {
                                    ADD_REPO = 'gremlins'
                                } else if ( repoName2.contains('core') ) {
                                    ADD_REPO = 'system'
                                } else if ( repoName2.contains('extra') ) {
                                    ADD_REPO = 'world'
                                }
                            }

                            if ( pkgStatus[0] == 'D' ) {
                                IS_REMOVE = 'true'
                                if ( repoName1.contains('staging') ) {
                                    RM_REPO = 'goblins'
                                } else if ( repoName1.contains('testing') ) {
                                    RM_REPO = 'gremlins'
                                } else if ( repoName1.contains('core') ) {
                                    RM_REPO = 'system'
                                } else if ( repoName1.contains('extra') ) {
                                    RM_REPO = 'world'
                                }
                            } else if ( pkgStatus[1] == 'D' ) {
                                IS_REMOVE = 'true'
                                if ( repoName2.contains('staging') ) {
                                    RM_REPO = 'goblins'
                                } else if ( repoName2.contains('testing') ) {
                                    RM_REPO = 'gremlins'
                                } else if ( repoName2.contains('core') ) {
                                    RM_REPO = 'system'
                                } else if ( repoName2.contains('extra') ) {
                                    RM_REPO = 'world'
                                }
                            }

                            PKG_TRUNK = buildInfo1[0] + '/trunk'
                        }

                        if ( entryCount == 3 && pkgCount == 2 ) {
                            def pkgEntry = changedPkgStatus[0].split()
                            def pkgStatus = pkgEntry[0]
                            def buildInfo1 = pkgPath[0].tokenize('/')
                            def buildInfo2 = pkgPath[1].tokenize('/')
                            def repoName1 = buildInfo1[2]
                            def repoName2 = buildInfo2[2]

                            if ( pkgStatus.contains('R') ) {
                                IS_ADD = 'true'
                                IS_REMOVE = 'true'

                                if ( repoName1.contains('staging') && repoName2.contains('testing') ) {
                                    ADD_REPO = 'gremlins'
                                    RM_REPO = 'goblins'
                                } else if ( repoName1.contains('testing') && repoName2.contains('staging') ) {
                                    ADD_REPO = 'goblins'
                                    RM_REPO = 'gremlins'
                                }

                                if ( repoName1.contains('core') && repoName2.contains('testing')) {
                                    ADD_REPO = 'gremlins'
                                    RM_REPO = 'system'
                                } else if ( repoName1.contains('testing') && repoName2.contains('core')) {
                                    ADD_REPO = 'system'
                                    RM_REPO = 'gremlins'
                                }

                                if ( repoName1.contains('extra') && repoName2.contains('testing')) {
                                    ADD_REPO = 'gremlins'
                                    RM_REPO = 'world'
                                } else if ( repoName1.contains('testing') && repoName2.contains('extra')) {
                                    ADD_REPO = 'world'
                                    RM_REPO = 'gremlins'
                                }

                                if ( repoName1.contains('core') && repoName2.contains('extra')) {
                                    ADD_REPO = 'world'
                                    RM_REPO = 'system'
                                } else if ( repoName1.contains('extra') && repoName2.contains('core')) {
                                    ADD_REPO = 'system'
                                    RM_REPO = 'world'
                                }
                            }
                            PKG_TRUNK = buildInfo1[0] + '/trunk'
                        }

                        if ( pkgCount == 1 ) {
                            def pkgEntry = changedPkgStatus[0].split()
                            def pkgStatus = pkgEntry[0]
                            def srcPath = pkgEntry[1]
                            def buildInfo = srcPath.tokenize('/')
                            def repoName = buildInfo[2]

                            if ( repoName.contains('staging') ) {
                                if ( pkgStatus == 'A' || pkgStatus == 'M' ) {
                                    IS_BUILD = 'true'
                                }
                                if ( pkgStatus == 'D' ) {
                                    IS_REMOVE = 'true'
                                }
                                ADD_REPO = 'goblins'
                                RM_REPO = ADD_REPO
                            } else if ( repoName.contains('testing') ) {
                                if ( pkgStatus == 'A' || pkgStatus == 'M' ) {
                                    IS_BUILD = 'true'
                                }
                                if ( pkgStatus == 'D' ) {
                                    IS_REMOVE = 'true'
                                }
                                ADD_REPO = 'gremlins'
                                RM_REPO = ADD_REPO
                            } else if ( repoName.contains('core') ) {
                                if ( pkgStatus == 'A' || pkgStatus == 'M' ) {
                                    IS_BUILD = 'true'
                                }
                                if ( pkgStatus == 'D' ) {
                                    IS_REMOVE = 'true'
                                }
                                ADD_REPO = 'system'
                                RM_REPO = ADD_REPO
                            } else if ( repoName.contains('extra') ) {
                                if ( pkgStatus == 'A' || pkgStatus == 'M' ) {
                                    IS_BUILD = 'true'
                                }
                                if ( pkgStatus == 'D' ) {
                                    IS_REMOVE = 'true'
                                }
                                ADD_REPO = 'world'
                                RM_REPO = ADD_REPO
                            }
                            PKG_PATH = srcPath
                            PKG_TRUNK = buildInfo[0] + '/trunk'
                        }

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
            }
            steps {
                dir("${PKG_PATH}") {
                    sh "buildpkg -r ${ADD_REPO}"
                }
            }
            post {
                success {
                    dir("${PKG_PATH}") {
                        sh "deploypkg -a -d ${ADD_REPO} -s"
                    }
                }
            }
        }
        stage('Add') {
            when {
                expression { return  IS_ADD == 'true' }
            }
            steps {
                dir("${PKG_TRUNK}") {
                    sh "deploypkg -a -d ${ADD_REPO}"
                }
            }
        }
        stage('Remove') {
            when {
                expression { return  IS_REMOVE == 'true' }
            }
            steps {
                dir("${PKG_TRUNK}") {
                    sh "deploypkg -r -d ${RM_REPO}"
                }
            }
        }
    }
}
