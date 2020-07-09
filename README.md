# Get started with Jenkins CI/CD in RedHat Openshift 4

```
oc version 
Client Version: 4.4.10
Server Version: 4.3.3
Kubernetes Version: v1.16.2

```

## Create project pipelineproject

```
oc new-project pipelineproject

```

## Search for Jenkins template
```
oc get templates -n openshift | grep jenkins 
jenkins-ephemeral                               Jenkins service, without persistent storage....                                    8 (all set)       6
jenkins-ephemeral-monitored                     Jenkins service, without persistent storage....                                    9 (all set)       7
jenkins-persistent                              Jenkins service, with persistent storage....                                       10 (all set)      7
jenkins-persistent-monitored                    Jenkins service, with persistent storage....                                       11 (all set)      8

```

## View template

```
oc get template/jenkins-ephemeral -o json -n openshift

```

## Process all parameters for a given openshift template 

```
oc process --parameters  -n openshift  jenkins-ephemeral # jenkins-persistent
NAME                              DESCRIPTION                                                                                                                                                                                                           GENERATOR           VALUE
JENKINS_SERVICE_NAME              The name of the OpenShift Service exposed for the Jenkins container.                                                                                                                                                                      jenkins
JNLP_SERVICE_NAME                 The name of the service used for master/slave communication.                                                                                                                                                                              jenkins-jnlp
ENABLE_OAUTH                      Whether to enable OAuth OpenShift integration. If false, the static account 'admin' will be initialized with the password 'password'.                                                                                                     true
MEMORY_LIMIT                      Maximum amount of memory the container can use.                                                                                                                                                                                           1Gi
NAMESPACE                         The OpenShift Namespace where the Jenkins ImageStream resides.                                                                                                                                                                            openshift
DISABLE_ADMINISTRATIVE_MONITORS   Whether to perform memory intensive, possibly slow, synchronization with the Jenkins Update Center on start.  If true, the Jenkins core update monitor and site warnings monitor are disabled.                                            false
JENKINS_IMAGE_STREAM_TAG          Name of the ImageStreamTag to be used for the Jenkins image.                                                                                                                                                                              jenkins:2
JENKINS_UC_INSECURE               Whether to allow use of a Jenkins Update Center that uses invalid certificate (self-signed, unknown CA). If any value other than 'false', certificate check is bypassed. By default, certificate check is enforced.                       false

```

## Deploy Jenkins using jenkins-ephemeral template

```
oc new-app jenkins-ephemeral
--> Deploying template "openshift/jenkins-ephemeral" to project bookinfo

     Jenkins (Ephemeral)
     ---------
     Jenkins service, without persistent storage.
     
     WARNING: Any data stored will be lost upon pod destruction. Only use this template for testing.

     A Jenkins service has been created in your project.  Log into Jenkins with your OpenShift account.  The tutorial at https://github.com/openshift/origin/blob/master/examples/jenkins/README.md contains more information about using this template.

     * With parameters:
        * Jenkins Service Name=jenkins
        * Jenkins JNLP Service Name=jenkins-jnlp
        * Enable OAuth in Jenkins=true
        * Memory Limit=1Gi
        * Jenkins ImageStream Namespace=openshift
        * Disable memory intensive administrative monitors=false
        * Jenkins ImageStreamTag=jenkins:2
        * Allows use of Jenkins Update Center repository with invalid SSL certificate=false

--> Creating resources ...
    route.route.openshift.io "jenkins" created
    deploymentconfig.apps.openshift.io "jenkins" created
    serviceaccount "jenkins" created
    rolebinding.authorization.openshift.io "jenkins_edit" created
    service "jenkins-jnlp" created
    service "jenkins" created
--> Success
    Access your application via route 'jenkins-bookinfo.apps.cluster-8faf.8faf.sandbox1706.opentlc.com' 
    Run 'oc status' to view your app.

```

## Verify the depliyment is completed. 

```
oc get all
NAME                   READY   STATUS      RESTARTS   AGE
pod/jenkins-1-deploy   0/1     Completed   0          96s
pod/jenkins-1-xbpcm    1/1     Running     0          87s

NAME                              DESIRED   CURRENT   READY   AGE
replicationcontroller/jenkins-1   1         1         1       96s

NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
service/jenkins        ClusterIP   172.30.157.171   <none>        80/TCP      97s
service/jenkins-jnlp   ClusterIP   172.30.134.17    <none>        50000/TCP   97s

NAME                                         REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfig.apps.openshift.io/jenkins   1          1         1         config,image(jenkins:2)

NAME                               HOST/PORT                                                        PATH   SERVICES   PORT    TERMINATION     WILDCARD
route.route.openshift.io/jenkins   jenkins-jenkins.apps.cluster-8faf.8faf.sandbox1706.opentlc.com          jenkins    <all>   edge/Redirect   None

```

## Setup Jenkins job
* We need to set up a pipeline to build our software, but we want to use the build that is built into OpenShift. The following command will create a build configuration (or “build config,” which is an object of type “BuildConfig”), which has the instructions we give to OpenShift to tell it how to build our application. In this particular case, we’re creating a pipeline that, in turn, has the build instructions:

```
oc create -f https://raw.githubusercontent.com/openshift/origin/master/examples/jenkins/pipeline/nodejs-sample-pipeline.yaml
```

## List builds

```
oc get buildconfigs 
NAME                     TYPE              FROM   LATEST
nodejs-mongodb-example   Source            Git    1
nodejs-sample-pipeline   JenkinsPipeline          1

```

## Start build

```
oc start-build nodejs-sample-pipeline
```

## Understanding build configurations
* https://docs.openshift.com/container-platform/4.4/builds/build-strategies.html#builds-tutorial-pipeline_build-strategies
* https://docs.openshift.com/container-platform/4.4/builds/understanding-buildconfigs.html

## Note: 
* OpenShift Pipelines Now Available as Technology Preview - https://www.openshift.com/blog/openshift-pipelines-tech-preview-blog 
* Creating Pipelines with OpenShift 4.4’s new Pipeline Builder and Tekton Pipelines - https://developers.redhat.com/blog/2020/04/30/creating-pipelines-with-openshift-4-4s-new-pipeline-builder-and-tekton-pipelines/

## Resources: 
* Using build strategies  - https://docs.openshift.com/container-platform/4.4/builds/build-strategies.html#builds-tutorial-pipeline_build-strategies
* Using templates 	  - https://access.redhat.com/documentation/en-us/openshift_container_platform/4.4/html/images/using-templates
* Build Strategy Tutorial - https://docs.openshift.com/container-platform/4.4/builds/build-strategies.html#builds-tutorial-pipeline_build-strategies
