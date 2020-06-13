# CICD Pipeline for API Connect via Jenkins using OpenShift

## Overview
In the [previous post](https://github.com/abhinabsarkar/apiconnect-pipeline-jenkins), it was explained how the Jenkins Distributed Architecture can be used to automate the API life cycle management in [API Connect](https://www.ibm.com/support/knowledgecenter/en/SSFS6T/com.ibm.apic.overview.doc/api_management_overview.html). It was just a tip of the iceberg to show how CICD can automate and improve your velocity to deliver, test, re-test, deploy and release software faster. Add a touch of [Containers](https://www.docker.com/what-container), [Docker](https://docs.docker.com/engine/docker-overview/) and [Orchestration](https://www.redhat.com/en/topics/containers/what-is-kubernetes) in the mix, and now you have added [acceleration scalability, density and consistent delivery](https://docs.docker.com/engine/docker-overview/#what-can-i-use-docker-for) of your applications.

Although this example covers CICD for API Connect – [IBM’s API Management Solution for Cloud and On-Premise](https://www.ibm.com/support/knowledgecenter/en/SSFS6T/com.ibm.apic.overview.doc/api_management_overview.html) using Jenkins and Openshift - [Container application platform by RedHat](https://www.openshift.com/) , this can be leveraged for any technology stack be it applications built on Java, .Net Core, Node js, etc. The message is **Hybrid is the new Cloud**.

## Architecture
Many organizations may already have an existing Jenkins Distributed Infrastructure running on Virtual Machines to run their continuous integration and continuous delivery pipelines, but they may still want to take advantage of the elasticity provided by Docker Container.

![Alt text](https://github.com/abhinabsarkar/apiconnect-pipeline-jenkins-openshift/blob/master/images/Architecture.png)

The architecture diagram here replaces the Jenkins Slave Agent VMs with containers running on RedHat’s OpenShift platform. A Docker image is built on top of [Jenkins Slave base image](https://github.com/openshift/jenkins/tree/master/slave-base), adding API Connect toolkit with yum install. This image is then made available as an image stream in OpenShift. That’s all that is needed on OpenShift Container Platform.
The Jenkins Master communicates with the OpenShift API and dynamically provisions the Jenkins Slave pod. This pod will have the container run inside it with API Connect toolkit. All this is functionality on the Jenkins to enable the dynamic capabilities between Jenkins Master and OpenShift is done by the use of Jenkins plugins. As simple as that.

The CICD deployment remains the same from here on.
1.	Pipeline is initiated on the Jenkins Master, which is delegated to the Jenkins Slave Pod in OpenShift.
2.	 The artifacts (Jenkinsfile, API configuration yaml files) are then pulled from the source control.
3.	API Connect toolkit publishes the artifacts to API Connect Dev environment and the automated tests are executed.
4.	Once the test cases are passed, API Connect toolkit publishes the artifacts to API Connect Test environment and the automated test cases are executed for Test environment. The same cycle is done through the multiple environments available.
5.	The deployment to Production environment is gated using change management.

## Code
The code for the Docker image and the image stream to run on OpenShift can be found [here](https://github.com/abhinabsarkar/apiconnect-pipeline-jenkins-openshift/tree/master/src)

## Jenkins Plugin
The list of plugins that are needed to be installed in the Jenkins Master environment are:
1. [Swarm](https://wiki.jenkins-ci.org/display/JENKINS/Swarm+Plugin) (Self-Organizing Swarm Plug-in Modules)
2. [Kubernetes](https://wiki.jenkins-ci.org/display/JENKINS/Kubernetes+Plugin)

## Jenkins Configuration
To completely disable jobs from executing on the master and to solely utilize slave, the executors count on the master needs to be set to 0. This value is set on the Jenkins system configuration page which can be accessed by selecting **Manage Jenkins** on the left-hand size of the master landing page and selecting **Configure System**. Next to **# of executors**, enter 0 and then hit **Save** to apply the changes.

Next, let’s configure the settings for the Kubernetes plugin on the system configuration page. Once again it can be accessed by selecting **Manage Jenkins** on the landing page and then selecting Configure System. Scroll down towards the bottom of the page and locate the **Cloudsection**.

Select **Add a new cloud** and choose **Kubernetes** which will generate a new section of configuration options.
Under Kubernetes, enter a name for the configuration, such as OpenShift and then enter the url of the OpenShift API.  You can choose to enter the server certificate key that is used for HTTPS communication or select the Disable HTTPS Security Check to disable this feature. Since a Kubernetes namespace is equivalent to an OpenShift project, enter jenkins in this text box next to the Kubernetes namespace field.

![Alt text](https://github.com/abhinabsarkar/apiconnect-pipeline-jenkins-openshift/blob/master/images/JenkinsConfigurationCloud.png)

Click the **Add** button to start the credential creation process and then in the dialog box next to kind, use the dropdown to select **OpenShift OAuth Access Token**. Enter the username, password and description into the textboxes. Finally click Add to create the credential. Select the credential from the dropdown box next to Credentials. Validate the master is able to successfully communicate with the OpenShift API by clicking the Test Connection button which should return a Connection Successful message.
When a new slave instance is provisioned by the master, it communicates with OpenShift to create a pod with the appropriate Docker image and settings to execute the job. These configurations are specified by defining a new Pod Template.
In the Kubernetes plugin configuration, select **Add Pod Template** and then **Kubernetes Pod Template** to add the fields to the configuration page. Example is shown below.

![Alt text](https://github.com/abhinabsarkar/apiconnect-pipeline-jenkins-openshift/blob/master/images/JenkinsConfigurationK8Pod.png)

Click on **Add Container** and select **Container Template**. Add Name and Docker image in the fields.

![Alt text](https://github.com/abhinabsarkar/apiconnect-pipeline-jenkins-openshift/blob/master/images/JenkinsConfigurationContainers.png)

Click Save to apply the changes to complete the required configuration to provision dynamic slaves. To test this, start a new pipeline build with the node set as **apic-pipeline (Label set on Kubernetes Pod Template)** in the pipeline job. The job will run and the pod can be seen running on OpenShift. 