pipeline {
    agent any
    tools{
        oc 'oc'
        maven 'maven-3.6.3'
        jdk 'jdk11'
    }
    stages {
        stage('Fetching Git Repository') {
            steps {
                git url: GIT_URL, branch: BRANCH
            }
        }
        stage ('Artifactory configuration') {
            steps {
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "localhost-jfrog-server",
                    releaseRepo: "libs-release-local",
                    snapshotRepo: "libs-snapshot-local"
                )
            }
        }
        stage ('Maven Build') {
            steps {
                rtMavenRun (
                    tool: "maven-3.6.3",
                    pom: 'pom.xml',
                    goals: '-s settings.xml clean install',
                    deployerId: "MAVEN_DEPLOYER"
                )
            }
        }
        stage ('Running Unit Tests') {
            steps {
                rtMavenRun (
                    tool: "maven-3.6.3",
                    pom: 'pom.xml',
                    goals: '-s settings.xml test'
                )
            }
        }
        stage ('Uploading Artifacts to Artifactory') {
            steps {
                rtPublishBuildInfo (
                    serverId: "localhost-jfrog-server"
                )
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
