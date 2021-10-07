# OpenShiftPipeline
This tutorial aims at showcasing the OpenShift Pipeline feature which is based on Tekton.

Pre-Requisite
1. OpenShift version 4.8
2. Fork a sample nodejs application from https://github.com/leeyeekee/nodejs-ex.git

# OpenShift Pipeline Key Concepts

OpenShift Pipelines provide a set of standard Custom Resource Definitions (CRDs) that act as the building blocks from which you can assemble a CI/CD pipeline for your application.

![image](https://user-images.githubusercontent.com/17938503/136389347-2045a6d2-5cb3-4020-a54f-dc3ea94a984a.png)

Task

A Task is the smallest configurable unit in a Pipeline. It is essentially a function of inputs and outputs that form the Pipeline build. It can run individually or as a part of a Pipeline. A Pipeline includes one or more Tasks, where each Task consists of one or more steps. Steps are a series of commands that are sequentially executed by the Task.

Pipeline

A Pipeline consists of a series of Tasks that are executed to construct complex workflows that automate the build, deployment, and delivery of applications. It is a collection of PipelineResources, parameters, and one or more Tasks. A Pipeline interacts with the outside world by using PipelineResources, which are added to Tasks as inputs and outputs.

PipelineRun

A PipelineRun is the running instance of a Pipeline. A PipelineRun initiates a Pipeline and manages the creation of a TaskRun for each Task being executed in the Pipeline.

Trigger

A Trigger captures an external event, such as a Git pull request and processes the event payload to extract key pieces of information. This extracted information is then mapped to a set of predefined parameters, which trigger a series of tasks that may involve creation and deployment of Kubernetes resources. You can use Triggers along with Pipelines to create full-fledged CI/CD systems where the execution is defined entirely through Kubernetes resources.

# OpenShift Pipeline Tutorial Demonstration

![image](https://user-images.githubusercontent.com/17938503/136392567-3525f4cc-1f88-4849-88fb-e055f284f27f.png)

Steps
1. Deploy the OpenShift Pipeline Operator
2. Create MyDev and MyStage project namespaces 
3. Grant ServiceAccounts to the project namespaces

        oc adm policy add-role-to-group edit system:serviceaccounts -n mydev

        oc admpolicy add-role-to-group edit system:serviceaccounts -n mystage

        oc adm policy add-role-to-user system:image-puller system:serviceaccounts:mystage -n mydev

        oc adm policy add-role-to-user system:deployer system:serviceaccounts:mydev -n mystage

5. Deploy the nodejs application with a OpenShift Pipeline onto the MyDev project namespace
6. Extend the pipeline to deploy to the MyStage project namespace
        Add a pipeline tasks of type openshift-client to tag the image and deploy to MyStage project namespace 

        Display Name : tag-image

        Script : 

          oc tag mydev/$(params.APP_NAME):latest mydev/$(params.APP_NAME):promote-stage

        Display Name : deploy-stage

        Script : 

          oc project mystage

          oc delete all --selector app=$(params.APP_NAME)

          oc new-app mydev/$(params.APP_NAME):promote-stage -n mystage --as-deployment-config

          oc scale --replicas=3 dc $(params.APP_NAME)

          oc expose service $(params.APP_NAME)

7. Create a pipeline trigger when a developer commit and push the source code changes

      - Add a pipeline trigger based on Git Push action

      - Copy pipeline trigger URL

      - Create a GitHub WebHook using the pipeline trigger URL

      - Update and commit the source code changes

      - Observe the triggering of the pipeline run

# Tutorial Demonstration Video

