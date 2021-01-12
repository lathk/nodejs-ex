        def templatePath = 'nodejs-mongodb-example'
        // name of the template that will be created
        def templateName = 'nodejs-mongodb-example'
        // NOTE, the "pipeline" directive/closure from the declarative pipeline syntax needs to include, or be nested outside,
        // and "openshift" directive/closure from the OpenShift Client Plugin for Jenkins.  Otherwise, the declarative pipeline engine
        // will not be fully engaged.
        pipeline {
            agent {
              node {
                // spin up a node.js slave pod to run this build on
                label 'nodejs'
              }
            }     
            
            options {
                // set a timeout of 20 minutes for this pipeline
                timeout(time: 5, unit: 'MINUTES')
            }

            stages {
                stage('build') {
                    steps {
                        script {
                            openshift.withCluster() {
                                openshift.withProject() {
                                    // create a new build from the templateName//
                                    // openshift.start-build(templateName)
                                       openshift.selector("bc", templateName).related('builds')
                                }
       //                             {
       //                             def builds = openshift.selector("bc", templateName).related('builds')
       //                             builds.untilEach(1) {
       //                                 return (it.object().status.phase == "Complete")
       //                             }
      //                          }
                            }
                        } // script
                    } // steps
                } // stage
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
            } // stages
        } //pipeline
      
