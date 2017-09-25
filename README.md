Amazon Simple Queue Service Ansible Playbook Bundle (SQS APB)
===============================================================
[Ansible Playbook Bundle (APB)](https://github.com/ansibleplaybookbundle/ansible-playbook-bundle) implementation of the [Amazon Simple Queue Service (SQS)](https://aws.amazon.com/sqs/).  This APB is implements two plans, Standard and FIFO.

## What it does
Deploys AWS SQS in OpenShift via CloudFormation Template.

## Requirements
* Openshift v3.7 or later with [Service Catalog](https://github.com/kubernetes-incubator/service-catalog) & [Ansible Service Broker](https://github.com/openshift/ansible-service-broker)
* AWS account ACCESS and SECRET Key pair

## Demo Video
Visit [this link](https://youtu.be/TczKwab0oMI) to view the demonstration of the SQS APB deployment and testing of its functionality in OpenShift

## Build the APB and Setup the OpenShift Environment
1. Create an OpenShift Environment with the [Service Catalog](https://github.com/kubernetes-incubator/service-catalog) & [Ansible Service Broker](https://github.com/openshift/ansible-service-broker).  You can use [CatASB](https://github.com/fusor/catasb), or you can skip this step if you already have an OpenShift Environment.
1. Download and install [APB Tool](https://github.com/ansibleplaybookbundle/ansible-playbook-bundle#installing-the-apb-tool)
1. Issue the `apb build` command in the APB's base directory (containing `apb.yml`)
1. Register your APB with the Service Catalog
    * Edit the broker's configmap via WebUI or by running the `oc edit configmap -n ansible-service-broker` command.
        Search for the following section of the configmap and ...
        ```
            broker:
            dev_broker: True
            bootstrap_on_startup: true
            refresh_interval: "600s"
        ```
        Edit, so that the `refresh_interval` looks like
        ```
            broker:
            dev_broker: True
            bootstrap_on_startup: true
            refresh_interval: "24h"
        ```
        Then redeploy the broker pod in WebUI or by running the `oc rollout latest asb -n ansible-service-broker` command to pick up the configmap changes.

        The above changes ensure that for the 24 hours after you run APB push, testing the APB you've pushed should continue to work without having to continually re-push every 10 minutes.
    * `apb push` the built APB image to your docker org/registry where the cluster's catalog/broker is configured for. 
    * Wait for the service catalog to refresh the APB list to show up in the WebUI. To force the refresh of the APB's, you can relaunch the 'controller-manager' pod of the service catalog, so that you won't have to wait the default time to refresh.

## Deployment
Once the APB is available in the WebUI you can deploy the APB
1. Create a new project for the APB
1. Select one of the two plans available (Standard or FIFO)
1. Enter the parameter values
1. Select 'do not bind at this time' option, and deploy
1. Review the logs of the APB and verify that the ansible playbook finsihed without any failures. 

## Verification
### AWS Console
After verifying that the APB provision successfully from the OpenShift's WebUI, you will still need to verify that the SQS creation was successful in AWS by logging in to the AWS console.  Address any errors on the AWS console (permissions, resource limitations, etc.), and redeploy the APB in OpenShift until the SQS creation is verified in the AWS console.

### Simple Test Application
When you have verified that the SQS was successfully created, you can deploy a simple test app in OpenShift to verify the SQS functionality.
1. Create a new [source-to-image](https://github.com/openshift/source-to-image) python app in Openshift by selecting `languages->python`
1. Select the project in which the SQS APB was deployed to
1. Select the `advanced options`, and do the following:
    * Enter the name of the test app (e.g. testapp)
    * Enter the git repo URL [https://github.com/johnkim76/aws-sqs-demo](https://github.com/johnkim76/aws-sqs-demo)
    * Besure the '`create a route to the application`' is checked
    * Click `create`
1. Go to the project and click on the URL of the test app
1. Verify that the link takes you to a page which says that the test app has not been binded yet
1. Create a new binding of your Test app to the SQS APB
1. Re-deploy/build the test app
1. Visit the test app URL again, and verify that the test app now provides a simple send & receive test links
1. Perform several sends and receives to check out the functionality of the SQS
