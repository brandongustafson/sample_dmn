# Table of Contents
<!--ts-->
   * [Sample DMN Project Deployment](#sample-dmn-Project)
   * [Openshift CI/CD Pipeline](#openshift-cicd-pipeline)
<!--te-->

# Sample-DMN-Project
- This repository contains the source code for a sample DMN Project which implements the iteration over list of objects on each of which business logic validation is performed.
- It can be deployed directly to a `Immutable Kie-server-v7.11.0` running on a openshift cluster configured for S2I(Source to Image).
## Prerequisites
- OpenShift v3.11 Cluster Web Console access with cluster-admin rights<br /><br />
## Installation
### Step 1. Download required files
 - Open this [link](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?downloadType=distributions&product=rhdm&version=7.11.0) in new tab which provides all the required download links.
 - Download the `Red Hat Decision Manager 7.11.0 OpenShift Templates` .zip (rhdm-7.11.0-openshift-templates.zip) file.
 - Unzip the downloaded zip file to a folder

 ### Step 2. Setting up the OpenShift Cluster
 - Login to your OpenShift Cluster Web Console or Terminal and create a new project
 - Create the following 2 secrets in the project namespace<br /><br/>
```
credentials
	KIE_ADMIN_USER = kie-admin
	KIE_ADMIN_PWD = kie-redhat1!
```

```
kie-server-https
	keystore.jks
	
```
 - credentials is a simple literals based secret containing 2 key value pairs. Here username and password can be any valid text secret. Make sure to keep a note as it will be used to login to the kie-server.
 - key-server-https secret is created from the file keystore.jks which is a SSL encryption key generated using the keytool. [Click here](https://access.redhat.com/documentation/en-us/jboss_enterprise_application_platform/6.1/html-single/security_guide/index#Generate_a_SSL_Encryption_Key_and_Certificate) to learn more<br /><br />
 - Open the openshift project/namespace(only accessible by cluster-admin) and go to `Builds`&nbsp;&rarr;&nbsp;`Images` section.
 - Using the Import YAML utility under Add to Project usually found at top-right corder, paste in the YAML script from `rhdm711-image-streams.yaml` file in the pop-up. optionally the `rhdm711-image-streams.yaml` file can be uploaded as well. `rhdm711-image-streams.yaml` file can be found inside the `rhdm-7.11.0-openshift-templates.zip` file. Now click create.
 - This step will create 3 image streams inside the openshift namespace which is used to pull the base images required for running the S2I build.
 
 ### Step 3. Adding and Configuring the Project
 - Here, the Immutable Production Ready S2I Configured Kie-server is being deployed on the OpenShift cluster which will run as a containerized api and can be consumed by other microservices or by the end consumer.
 - In the overview section, use the Import YAML utility to add the project template and configure it. upload or paste in the YAML script from the `rhdm711-prod-immutable-kieserver.yaml` file which can be found in the r`hdm-7.11.0-openshift-templates.zip`&nbsp;&rarr;&nbsp;`templates` folder.
 - After clicking on create, make sure Process the template is selected in the propmt and then click continue.<br /><br />
![](https://github.com/brandongustafson/sample_dmn/blob/master/docs/Import.png)<br /><br />
 - Optionally the rhdm711-prod-immutable-kieserver.yaml can be modified following the below steps and resource can be created using the `oc create -f [yamlFile]` command.
 - Set the following parameters on the next screen(other parameters are optinal to configure, [Click here](https://access.redhat.com/documentation/en-us/red_hat_decision_manager/7.11/html/deploying_red_hat_decision_manager_on_red_hat_openshift_container_platform/environment-immutable-con_openshift-templates) to learn more):
 	- `Application Name (APPLICATION_NAME)` The name of the OpenShift application. It is used in the default URL for Decision Server. OpenShift uses the application name to create a separate set of deployment configurations, services, routes, labels, and artifacts.
 	- `Credentials secret (CREDENTIALS_SECRET)` The name of the secret containing the administrative user credentials. `credentials` in this case.
 	- `ImageStream Namespace (IMAGE_STREAM_NAMESPACE)` The namespace where the image streams are available. `openshift` in this case. Make sure the kie-server version is v7.11.0.
 	- `KIE Server Keystore Secret Name (KIE_SERVER_HTTPS_SECRET)` The name of the secret for KIE Server containing the SSL Key. `kie-server-https` in this case.
 	- `KIE Server Certificate Name (KIE_SERVER_HTTPS_NAME)` The name of the certificate in the keystore.
 	- `KIE Server Keystore Password (KIE_SERVER_HTTPS_PASSWORD)` The password for the keystore.<br /><br />
 - KIE Server Container Deployment (KIE_SERVER_CONTAINER_DEPLOYMENT): The identifying information of the decision service (KJAR file) that the deployment must pull from the local or external repository after building your source. The format is `containerId=groupId:artifactId:version` or, if you want to specify an alias name for the container, `containerId(aliasId)=groupId:artifactId:version` You can provide two or more KJAR files using the | separator
 	- `IterationDemo_1.0.0-SNAPSHOT=com.myspace:IterationDemo:1.0.0-SNAPSHOT` in this case.<br /><br />
![](https://github.com/brandongustafson/sample_dmn/blob/master/docs/Config.png)
 	- The project specific information can be obtained by running a GET request at `http://localhost:8080/kie-server/services/rest/server/containers` &rarr;&nbsp;`project` object in a local development environment.
 	- Git Repository URL (SOURCE_REPOSITORY_URL): The URL for the Git repository that contains the source code of the service.<br />`https://github.com/brandongustafson/sample_dmn.git in this case.`<br /><br />
 - Set CPU limit to at least 2 to make sure at least 1 pod is active.<br /><br />
![](https://github.com/brandongustafson/sample_dmn/blob/master/docs/Confirm.png)
 - Click create anyway on the pop-up that shows up
 - After the build step is successfull, change the replica count to 1 in kie-server deployment configuration. This is to ensure that the microservice starts with 1 instance and then can be scaled up. More pods require more cpus to be used. Keep an eye for `Insufficient CPU` warning in the event logs if the pods are not scaling.

 ## Troubleshooting Guide
 - Errors might occur during the build step due to missing dependencies. In `Builds`&nbsp;&rarr;&nbsp;`Builds` Section in OpenShift Web Console, in the status column if it shows Assemble failed then there are possible version conflicts or something wrong with the source code.
 - Click on the build number in the entry that failed (has status Assemble Failed), latest build is at the top. Click on logs tab which will display the build logs. Generally there might be missing dependencies which will throw an build/compilation error and print out an error in the logs showing the missing dependency.
 - Missing dependencies can added by modifying the `pom.xml` file which is present in the root directory of the source code repository, get the required xml tags by looking up the dependency name on maven repository and selecting the version that is used by other dependencies which can be found in the logs when the Downloading step occurs.
 - After the missing dependencies are added and the changes committed to source code repository, click build. This will cause the project to go through another build process.<br /><br />
 - If the build is successfull but the pods are staying in pending state, are not ready or being constantly restarted, check the event logs of the project which can be found in the Monitoring section.
 - If there is a `Insufficient CPU` warning, it means the project doesn't have enough processing/cpu resources. Change the cpu configuration in the kie-server deployment configuration by editing cpu-limit in the YAML configuration to at least 2. deployment configuration is found under `Applications`&nbsp;&rarr;&nbsp;`Deployments` Section in OpenShift Web Console.

## Payload Configuration
- After the kie server project is deployed and configured on the OpenShift cluster, services and routes are created which exposes the kie-server REST API.Examples are shown below as how to access various REST Enpoints.
- Swagger UI can be accessed using the following URL<br />`GET http://{api-url}/docs`<br /><br />
- Execute the following GET Request under the Kie Server and Kie Containers section to retrieve the **container Id**.<br />`GET http://{api-url}/services/rest/server/containers`<br /><br />
![](https://github.com/brandongustafson/sample_dmn/blob/master/docs/GET%20Containers.png?raw=true)<br /><br />
- Execute the following GET Request under the DMN models section to retrieve the **model-namespace** and **model-id** by passing in the container Id, set the Response content type to application/json.<br />`GET http://{api-url}/services/rest/server/containers/IterationDemo_1.0.0-SNAPSHOT/dmn`<br /><br />
![](https://github.com/brandongustafson/sample_dmn/blob/master/docs/GET%20Info.png?raw=true)<br /><br />
- Execute the following POST Request to check for the validation by passing in the payload configured as shown in **Payload** in the body parameter, set the Parameter content type and Response content type to application/json.<br />`POST http://{api-url}/services/rest/server/containers/IterationDemo_1.0.0-SNAPSHOT/dmn`<br /><br />
![](https://github.com/brandongustafson/sample_dmn/blob/master/docs/POST%20Info.png?raw=true)<br /><br />
### Payload - Configurationn:
```
{
  "model-namspace": model-namespace,
  "model-name": model-name,
  "dmn-context":{
    "Input": [
        {"id": id, "value": value},
        {"id": id, "value": value},
        ...
      ]
    }
}
```
### Payload - Example:
`container Id: IterationDemo_1.0.0-SNAPSHOT`
<br />
```
{
  "model-namespace": "https://kiegroup.org/dmn/_5388429C-0B33-4FA1-95DD-FA7A69AF31E6",
  "model-name": "SampleDMN",
  "dmn-context":{
    "Input": [
        {"id": "1", "value": 1},
        {"id": "2", "value": 100},
        {"id": "2", "value": 100}
      ]
    }
}
```
### Payload - Expected Reponse:
```
{
  "type": "SUCCESS",
  "msg": "OK from container 'IterationDemo_1.0.0-SNAPSHOT'",
  "result": {
    "dmn-evaluation-result": {
      "messages": [],
      "model-namespace": "https://kiegroup.org/dmn/_5388429C-0B33-4FA1-95DD-FA7A69AF31E6",
      "model-name": "SampleDMN",
      "decision-name": [],
      "dmn-context": {
        "Input": [
          {
            "id": "1",
            "value": 1
          },
          {
            "id": "2",
            "value": 100
          },
          {
            "id": "2",
            "value": 100
          }
        ],
        "BusinessKnowledgeModel": "function BusinessKnowledgeModel( inputParam )",
        "Decision": [
          "<100",
          ">=100",
          ">=100"
        ]
      },
      "decision-results": {
        "_F6344A84-9E0E-4C4F-AC9E-1172AD846A36": {
          "messages": [],
          "decision-id": "_F6344A84-9E0E-4C4F-AC9E-1172AD846A36",
          "decision-name": "Decision",
          "result": [
            "<100",
            ">=100",
            ">=100"
          ],
          "status": "SUCCEEDED"
        }
      }
    }
  }
}
```
The business validation results are in the last `decision-results` object. `result` object contains the output of business logic validation performed over a list of input.



# Openshift-CI/CD-Pipeline

### Prerequisites
- JDK 11

### Setup
- **Step 1: Installing Jenkins on your local machine**
  - The latest version of Jenkins can be found [here](https://www.jenkins.io/). Download the Generic Java package (.war) file. Make sure java version 8 or higher is installed. Java version can be checked by using the following command on a terminal on the local machine:
 
		java --version
    If you do not have JDK installed, the following link can be used
   <b>JDK 11</b>: [link](https://www.oracle.com/java/technologies/javase-downloads.html#JDK11)
- **Step 2: Starting the Jenkins Automation tool**
  - Run the following command to cd into the Downloads folder (or where your jenkins.war file is):

		cd ~/Downloads

  - Run the following command to start the Jenkins Application
		java -jar jenkins.war
		
- **Step 3: Setting up Jenkins**
  - Go to [localhost:8080](http://localhost:8080) to see the homepage of Jenkins, enter the admin password at which point the Jenkins configuration will start. It's a simple one-time admin configuration. Select install suggested plug-ins to get started quickly.

- **Step 4: Installing plugins**
  - Plugins can be installed via **Dashboard**&nbsp;&rarr;&nbsp;**Manage Jenkins**&nbsp;&rarr;&nbsp;**Manage Plugins**
  - Click on "Available" tab and search the following plug-ins
  - To work with OpenShift, following plug-ins are required:
    - OpenShift Client
    - OpenShift Login
    - OpenShift Sync

  - Artifactory Plug-ins:
    - Artfactory

  - To add the functionality of having multiple users with roles and permissions, install:
    - Role-based Authorization Strategy

   Select the above plug-ins and hit **Install without Restart** button.
- **Step 5 Installing and Setting up JFrog Artifactory Server:**

  - Download the OSS (Open Source Solution) version of Artifactory from [here](https://releases.jfrog.io/artifactory/bintray-artifactory/org/artifactory/oss/jfrog-artifactory-oss/[RELEASE]/jfrog-artifactory-oss-[RELEASE]-darwin.tar.gz)
  - Unzip the tar file by double clicking on the downloaded tar.gz file or using the following command:
 `tar -xvf [zip fileName]`
  - After the file is unzipped, open up a terminal and cd into
 `artifactory-oss/app/bin/`
  - If the file is unzipped into the downloads folder, use the following command to cd into the folder:
 `cd ~/Downloads/artifactory-oss-7.21.12/app/bin/`
  - Once you are in the bin folder, run the following command to start the Artifactory server
 `./artifactory.sh start`
  - If you get a malware prompt:
    - Open settings
    - Select security & privacy settings
    - In the general tab, click the lock near the bottom left
    - Note that the “allowed apps” list will update as you run the ./artifactory.sh start command
    - While the program is running, you will be prompted to move to trash, or close. Click close then look at the “allowed apps” list in your settings.  A new entry will appear each time you hit close.  Click “always allow” then restart the ./artifactory.sh start command.  Repeat this process until the installation is complete.

  - It should start the artifactory server on the following URL : [http://localhost:8082](http://localhost:8082). Wait for artifactory to start and be ready to serve

```
Default login credentials:
	Username: admin
	Password: password
```

- **Step 6 Setting up Artifactory server:**
  - Step 1: Set up the admin password (Optional, you can skip if you want to use the default password). Select any easy password that you can remember, for example *Admin123!*
  - Step 2: Set Base URL
    - Skip
  - Step 3: Configure Proxy Server
    - Skip
  - Step 4: Create a Repository/ Repositories (if it shows up)
    - Skip

- **Step 7 Creating Maven Repositories on JFrog Artifactory Server**
  - Go to Administration section(Gear icon in the left navigation pane, on the top)
  - Click on Repositories&nbsp;&rarr;&nbsp;Repositories&nbsp;&rarr;&nbsp;Local (Local tab in the main area)<br />
  - Click Add Repositories from top right corner,
    - Select Local Repository
    - Select Maven
    - Enter Repository key as : libs-release-local
    - Click Save & Finish in the bottom right corner

  - Click Add Repositories from top right corner,
    - Select Local Repository
    - Select Maven
    - Enter Repository key as : libs-snapshot-local
    - Click Save & Finish in the bottom right corner

   **NOTE: Use these credentials when configuring JFrog server in Jenkins**:

   Go to **Administration**&nbsp;&rarr;&nbsp;**Identity and Access**&nbsp;&rarr;&nbsp;**Users**
  - Click on “New User” in the top right corner
    - Username: { username } (example: deployer)
    - Email: { email } (example: deployer@artifactory.com)
    - Roles : Administrator Platform - Checked
    - Password: { password } (example: Password123!)
    - Click Save in the bottom right corner

- **Step 8 Configure Jenkins Global Tool Configuration**
  - This step is to configure tools such as java, maven and openshift client (oc) to be used by pipelines. Tools name given under these settings will be called under the pipeline tool section.

  - Go to Manage Jenkins&nbsp;&rarr;&nbsp;Global Tool Configuration

    - OpenShift Client Tools
      - Click on "Add OpenShift Client Tools"
      - Name : oc
      - Install automatically : checked
      - Click on "Add OpenShift Client Tools
      - Choose "Extract *.zip/*.tar.gz"
      - Label : Leave empty
      - Download URL for binary archive : https://mirror.openshift.com/pub/openshift-v4/clients/oc/latest/macosx/oc.tar.gz
    ** IMPORTANT: Use oc cli whose version matches the openshift cluster**
      - Subdirectory of extracted archive : Leave empty

	  Click **Apply**

    - JDK Installation
      - Click on "Add JDK"
      - Name : jdk11
      - Install Automatically : uncheck
      - JAVA_HOME :
      Use command `echo $JAVA_HOME` which will print out the required path
	  /Library/Java/JavaVirtualMachines/jdk-11.0.12.jdk/Contents/Home/

	  Click **Apply**

    - Maven Installation
      - Click on "Add Maven"
      - Name : maven-3.6.3
      - Install automatically : checked
      - Install from Apache Version : 3.6.3

     Click **Apply & Save**

- **Step 9 : Jenkins Configurations : Configure OpenShift Cluster Details**
   - Go to Manage Jenkins -> Configure System

   - OpenShift Client Plugin
      - Cluster Configurations : Click on "Add OpenShift Cluster"
      - Cluster Name : openshift-cluster
      - API Server URL : *yourOcpClusterUrl*
      - Disable TLS Verify : checked
      - Credentials : Click on "Add"
      - Kind : "OpenShift Token for OpenShift Client Plugin"
      - ID : {unique Id}
      - Click "Add"
      - Credentials : Choose from drop down {unique id}

   Click on **Apply & Save**

   - JFrog
      - Cluster Configurations : Click on “Add JFrog Platform Instance”
      - Instance Id : localhost-jfrog-server
      - JFrog Platform URL : { localhost-url } (generally http://localhost:8082)
      - Default Deployer Credentials:
      - Username : { JFrog Login Username }
      - Password : { JFrog Login Password }

   Click on **Test Connection** and see if it works, if it does then hit **Apply and Save**

- **Step 10 : Create an OpenShift Pipeline**
  - The pipeline will use parameters to capture git repo, branch, project namespace, openshift cluster name (as configured above earlier) and the Build Configuration.

  - Open Jenkins&nbsp;&rarr;&nbsp;Dashboard &nbsp;&rarr;&nbsp;New Item

  - Creating and Configuring the Pipeline

   - Enter an item name : OpenShift Pipeline
     - Select "Pipeline" & Click "OK"
     - Under General : Check / Select "This project is parameterized"

   - Click on "Add Parameter" & Choose "String Parameter"
     - Name : GIT_URL
     - Default-Value : this git repo

   - Click on "Add Parameter" & Choose "String Parameter"
     - Name : BRANCH
     - Default-Value : master

   - Click on "Add Parameter" & Choose "String Parameter"
     - Name : CLUSTER_NAME
     - Default Value : openshift-cluster
**IMPORTANT - This should match with earlier defined cluster name**

   - Click on "Add Parameter" & Choose "String Parameter"
     - Name : PROJECT_NAME
     - Default Value : sample-dmn-demo

   - Click on "Add Parameter" & Choose "String Parameter"
     - Name : BUILD_CONFIG
     - Default Value : myappsample-iteration-demo-kieserver

   - Click on "Add Parameter" & Choose "String Parameter"
     - Name : DEPLOYMENT_CONFIG
     - Default Value : myappsample-iteration-demo-kieserver

  - Under Pipeline Section
     - Select Pipeline from SCM in the definition section.
     - Select Git in SCM
     - Enter this git repo url in the github URL
     - Credentials : none (for public repositories)

     - ScriptPath: Jenkinsfile
     - Lightweight Checkout : selected

   Hit **Apply & Save**

Run the pipeline:

 - Click on Build with Parameters
  - GIT_URL : https://github.com/brandongustafson/sample_dmn/sample-dmn-iteration.git
  - BRANCH : master
  - PROJECT_NAME : sample-dmn-demo
  - CLUSTER_NAME : openshift-cluster
  - BUILD_CONFIG : myappsample-iteration-demo-kieserver

Click on **Build**
