pipeline {
    agent {
      node {
        label 'nodejs'
      }
    }
    stages {
        stage('preamble') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject() {
                            echo "Using project: ${openshift.project()}"
                        }
                    }
                }
            }
        }
        stage('cleanup') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject() {
                            openshift.selector("all", [ template : nodejs-mongodb-example ]).delete()
                            if (openshift.selector("secrets", nodejs-mongodb-example).exists()) {
                                openshift.selector("secrets", nodejs-mongodb-example).delete()
                            }
                        }
                    }
                } // script
            } // steps
        } // stage
        stage('create') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject() {
                            openshift.newApp(nodejs-mongodb-example)
                        }
                    }
                } // script
            } // steps
        } // stage
        stage('build') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject() {
                            def builds = openshift.selector("bc", nodejs-mongodb-example).related('builds')
                            builds.untilEach(1) {
                                return (it.object().status.phase == "Complete")
                            }
                        }
                    }
                } 
            } 
        } 
        stage('deploy') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject() {
                            def rm = openshift.selector("dc", nodejs-mongodb-example).rollout()
                            openshift.selector("dc", nodejs-mongodb-example).related('pods').untilEach(1) {
                                return (it.object().status.phase == "Running")
                            }
                        }
                    }
                } // script
            } // steps
        } // stage
        stage('tag') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject() {
                            openshift.tag("nodejs-mongodb-example:latest", "nodejs-mongodb-example-staging:latest")
                        }
                    }
                } // script
            } // steps
        } // stage
    } // stages
    options {
        timeout(time: 20, unit: 'MINUTES')
    }
} // pipeline
