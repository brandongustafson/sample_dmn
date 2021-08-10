# Sample DMN Project
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
![](https://github.com/rutvik-nvs/sample-dmn-iteration/blob/master/docs/Import.png)<br /><br />
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
![](https://github.com/rutvik-nvs/sample-dmn-iteration/blob/master/docs/Config.png)
 	- The project specific information can be obtained by running a GET request at `http://localhost:8080/kie-server/services/rest/server/containers` &rarr;&nbsp;`project` object in a local development environment.
 	- Git Repository URL (SOURCE_REPOSITORY_URL): The URL for the Git repository that contains the source code of the service.<br />`https://github.com/rutvik-nvs/sample-dmn-iteration.git in this case.`<br /><br />
 - Set CPU limit to at least 2 to make sure at least 1 pod is active.<br /><br />
![](https://github.com/rutvik-nvs/sample-dmn-iteration/blob/master/docs/Confirm.png)
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
![](https://github.com/RutvikPanchal/sampleDMN/blob/master/docs/GET%20Containers.png?raw=true)<br /><br />
- Execute the following GET Request under the DMN models section to retrieve the **model-namespace** and **model-id** by passing in the container Id, set the Response content type to application/json.<br />`GET http://{api-url}/services/rest/server/containers/IterationDemo_1.0.0-SNAPSHOT/dmn`<br /><br />
![](https://github.com/RutvikPanchal/sampleDMN/blob/master/docs/GET%20Info.png?raw=true)<br /><br />
- Execute the following POST Request to check for the validation by passing in the payload configured as shown in **Payload** in the body parameter, set the Parameter content type and Response content type to application/json.<br />`POST http://{api-url}/services/rest/server/containers/IterationDemo_1.0.0-SNAPSHOT/dmn`<br /><br />
![](https://github.com/RutvikPanchal/sampleDMN/blob/master/docs/POST%20Info.png?raw=true)<br /><br />
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
