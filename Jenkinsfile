pipeline {
    agent any
    tools{
        oc 'oc'
        maven 'maven-3.6.3'
        jdk 'jdk11'
    }
    

        stage ('Deploy Kieserver') {
            steps {
                script {
                    openshift.withCluster( CLUSTER_NAME ) {
                        openshift.withProject( PROJECT_NAME ){
                            def processedTemplate
                            
                           
                           if( PROJECT_NAME ){
                                 try {
                                    processedTemplate = openshift.process( "-f", "./docs/rhdm711-prod-immutable-kieserver.yaml", "--param-file=./docs/template-create.env")
                                    def createResources = openshift.create( processedTemplate )
                                    createResources.logs('-f')
                                    
                                 } catch (err) {
                                    echo err.getMessage()
                                }
                            } 
                         
                        }
                    }
                }
           }
        }

        stage ('Building and Pushing Image to Quay') {
            steps {
                script {
                    openshift.withCluster( CLUSTER_NAME ) {
                        openshift.withProject( PROJECT_NAME ){
                            def buildConfig = openshift.selector( 'buildconfig/' + BUILD_CONFIG )
                            buildConfig.startBuild()
                            buildConfig.logs('-f')
                        }
                    }
                }    
            }
        }
        stage ('Deploying') {
            steps {
                script {
                    openshift.withCluster( CLUSTER_NAME ) {
                        openshift.withProject( PROJECT_NAME ){
                            def devConfig = openshift.selector( 'deploymentconfig/' + DEPLOYMENT_CONFIG )
                            devConfig.rollout().latest()
                        }
                    }
                }    
            }
        }
    }
}
