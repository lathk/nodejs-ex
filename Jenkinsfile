def templatePath = 'nodejs-mongodb-example'
def templateName = 'nodejs-mongodb-example'

pipeline {
    agent {
      node {
        label 'nodejs'
      }
    }
    options {
        timeout(time: 20, unit: 'MINUTES')
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
                                    openshift.selector("all", [ template : templateName ]).delete()
                                    if (openshift.selector("secrets", templateName).exists()) {
                                        openshift.selector("secrets", templateName).delete()
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
                                    openshift.newApp(templatePath)
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
                                    def builds = openshift.selector("bc", templateName).related('builds')
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
                                    def rm = openshift.selector("dc", templateName).rollout()
                                    openshift.selector("dc", templateName).related('pods').untilEach(1) {
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
                                    // if everything else succeeded, tag the ${templateName}:latest image as ${templateName}-staging:latest
                                    // a pipeline build config for the staging environment can watch for the ${templateName}-staging:latest
                                    // image to change and then deploy it to the staging environment
                                    openshift.tag("${templateName}:latest", "${templateName}-staging:latest")
                                }
                            }
                        } // script
                    } // steps
                } // stage
    } // stages
} // pipeline
