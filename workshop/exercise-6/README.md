# To the terminal!

## Deploying an application via the `oc` CLI
In this section we'll show how to interact with your OpenShift cluster via the CLI. We'll be running through the same general steps as the main lab, just via the command line. 

### Install OpenShift CLI tools
Download and unpack OpenShift cli tools. The `oc` utility is your main gateway into OpenShift. We'll add them to your path in a convenient location. Please double check the latest release here: [OpenShift Origin Releases](https://github.com/openshift/origin/releases/)


Download tarball of the tools
```bash
wget https://github.com/openshift/origin/releases/download/v3.11.0/openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz
```

Unpack
```bash
tar -xvzf openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz
```

Rename for ease of use
```bash
mv openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit ${HOME}/oc-cli
```

Set Path.

```bash
export PATH=${PATH}:${HOME}/oc-cli
```

Verify the utility is available by using `which` and the help

```bash
which oc
```

```bash
oc help
```

## Access the OpenShift Web UI

1. Launch the OpenShift web console.
   - Navigate to the [IBM Cloud Clusters Dashboard](https://cloud.ibm.com/kubernetes/clusters)
   - Find your cluster and click it
   - Click `OpenShift Web Console` on the top right
2. Once in the console, select your name/id in the upper right, click. Scroll down to 'Copy Login Command' and click it.

![Copy oc login command](https://dsc.cloud/quickshare/Shared-Image-2019-09-25-08-38-04.png)

## Access your cluster using OpenShift client utils

1. Paste the login command you copied from the Web UI.

```bash
oc login https://c100-e.us-east.containers.cloud.ibm.com:39813 --token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

You should see a success message.

## Validate access to your cluster.
View nodes in the cluster.

```bash
oc get node
```

View services, deployments, and pods.

```bash
oc get svc,deploy,po --all-namespaces
```

View all OpenShift projects

```bash
oc get projects
```

### Deploy using Github repo and image stream 
You can use the `new-app` command and target both an image builder and a Source repository for your application code. In this case we're using a base OpenShift Node.js image with an example Health care app. 

```
$ oc new-app openshift/nodejs:10~https://github.com/greyhoundforty/health-app-s2i.git --name=v3
--> Found image 93de123 (11 months old) in image stream "openshift/nodejs" under tag "10" for "openshift/nodejs:10"

    Node.js 10.12.0
    ---------------
    Node.js  available as docker container is a base platform for building and running various Node.js  applications and frameworks. Node.js is a platform built on Chrome's JavaScript runtime for easily building fast, scalable network applications. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient, perfect for data-intensive real-time applications that run across distributed devices.

    Tags: builder, nodejs, nodejs-10.12.0

    * A source build using source code from https://github.com/greyhoundforty/health-app-s2i.git will be created
      * The resulting image will be pushed to image stream tag "v3:latest"
      * Use 'oc start-build' to trigger a new build
    * This image will be deployed in deployment config "v3"
    * Port 8080/tcp will be load balanced by service "v3"
      * Other containers can access this service through the hostname "v3"

--> Creating resources ...
    imagestream.image.openshift.io "v3" created
    buildconfig.build.openshift.io "v3" created
    deploymentconfig.apps.openshift.io "v3" created
    service "v3" created
--> Success
    Build scheduled, use 'oc logs -f bc/v3' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/v3'
    Run 'oc status' to view your app.
```

This will create an ImageStream, Deployment, a Pod, and a Service resource for the example Health app.

### Check on status 
We can check on the status of the application by using the `status` command. Throwing the `--suggest` flag will look at your current rollout and make suggestions on ways to fix any possible issues. In this case it is suggesting that we add a rediness and liveness probe. 

```
$ oc status --suggest
In project Health App v2 (health-app-v2) on server https://c100-e.us-south.containers.cloud.ibm.com:32284

svc/v3 - 172.21.125.223:8080
  dc/v3 deploys istag/v3:latest <-
    bc/v3 source builds https://github.com/greyhoundforty/health-app-s2i.git on openshift/nodejs:10
    deployment #1 deployed 8 seconds ago - 1 pod

Info:
  * dc/v3 has no readiness probe to verify pods are ready to accept traffic or ensure deployment is successful.
    try: oc set probe dc/v3 --readiness ...
  * dc/v3 has no liveness probe to verify pods are still running.
    try: oc set probe dc/v3 --liveness ...

View details with 'oc describe <resource>/<name>' or list everything with 'oc get all'.
```

### Expose service 
By default our application is only exposed as a service within the cluster itself. To add a public route we'll use the `expose` option. If no hostname is specified, a randomly generated one will be created. 

```
$ oc expose svc/v3
route.route.openshift.io/v3 exposed

$ oc get routes
NAME   HOST/PORT                                                                                            PATH   SERVICES   PORT       TERMINATION   WILDCARD
v3     v3-health-app-v2.roks-rt-007632d2d5235aabd90e420d6faed9fd-0001.us-south.containers.appdomain.cloud          v3         8080-tcp                 None
```

### Test site
We'll target the `/info` endpoint for our example health application:

```
$ curl -s v3-health-app-v2.roks-rt-007632d2d5235aabd90e420d6faed9fd-0001.us-south.containers.appdomain.cloud/info 
{
    "appointments": [
        "2018-01-15 1:00 - Dentist",
        "2018-02-14 4:00 - Internal Medicine",
        "2018-09-30 8:00 - Pediatry"
    ],
    "medications": [
        "Metoprolol",
        "ACE inhibitors",
        "Vitamin D"
    ],
    "personal": {
        "age": 38,
        "city": "Toronto",
        "gender": "male",
        "name": "Ralph DAlmeida",
        "street": "34 Main Street",
        "zipcode": "M5H 1T1"
    }
}
```

### Add Health Checks 
Just like in our main lab, we'll add some Health checks to our application using the `probe` command. A probe is a Kubernetes action that periodically performs diagnostics on a running container. 

```
$ set probe dc/v3 --readiness --get-url=http://:8080/info --initial-delay-seconds=30 --timeout-seconds=2
deploymentconfig.apps.openshift.io/v3 probes updated

$ oc set probe dc/v3 --liveness --get-url=http://:8080/info --initial-delay-seconds=30 --timeout-seconds=2
deploymentconfig.apps.openshift.io/v3 probes updated

$ oc status
In project Health App v2 (health-app-v2) on server https://c100-e.us-south.containers.cloud.ibm.com:32284

http://v3-health-app-v2.roks-rt-007632d2d5235aabd90e420d6faed9fd-0001.us-south.containers.appdomain.cloud to pod port 8080-tcp (svc/v3)
  dc/v3 deploys istag/v3:latest <-
    bc/v3 source builds https://github.com/greyhoundforty/health-app-s2i.git on openshift/nodejs:10
    deployment #3 running for 33 seconds - 1 pod
    deployment #2 deployed about a minute ago
    deployment #1 deployed 7 minutes ago
```

### Add Resource Limits 

```
$ oc set resources DeploymentConfigs/v3 --limits=cpu=30m,memory=100Mi --requests=cpu=3m,memory=40Mi
deploymentconfig.apps.openshift.io/v3 resource requirements updated
```

### Add Autoscaler 
You can create a horizontal pod autoscaler with the oc autoscale command and specify the minimum and maximum number of pods you want to run, as well as the CPU utilization or memory utilization your pods should target.

```
$ oc autoscale dc/v3 --min=2 --max=10 --cpu-percent=10
horizontalpodautoscaler.autoscaling/v3 autoscaled
```

## Add Github Webhook integration
So far we have been doing alot of manual deployment. In cloud-native world we want to move away from manual work and move toward automation. Wouldn't it be nice if our application rebuilt on git push events? Git webhooks are the way its done and openshift comes bundled in with git webhooks. Let's set it up for our project.

To be able to setup git webhook we need to have elevated permission to the project. The repo we have been using so far we don't own it. But since its opensource we can easily fork it and make it our own.

Fork the repo at [https://github.com/greyhoundforty/health-app-s2i](https://github.com/greyhoundforty/health-app-s2i)

![Fork](https://dsc.cloud/quickshare/Screen-Shot-2019-09-24-at-11.56.01-AM.png)

Now that I have forked the repo under my repo I have full admin priviledges. As you can see I now have a settings button that I can change the repo settings with.

![Forked Repo](https://dsc.cloud/quickshare/Shared-Image-2019-09-24-11-59-28.png)

We will come back to this page in a moment. For now let's jump back to our OpenShift console and change our applications git source to point to our repository.

From our openshift dashboard for our project. Select `Builds > Builds`

![Goto Build](https://dsc.cloud/quickshare/Shared-Image-2019-09-24-12-01-09.png)

Select the `v3` build (or whatever you named your app in the `oc new-app` command we ran previously). As of now this should be the only build on screen.

![Select Build](https://dsc.cloud/quickshare/Shared-Image-2019-09-24-12-03-20.png)

Click on `Action` on the right and then select `Edit`

![Edit Build](https://dsc.cloud/quickshare/Shared-Image-2019-09-24-12-05-39.png)

Change the `Git Repository URL` to our forked repository.

![New Repo](https://dsc.cloud/quickshare/Shared-Image-2019-09-24-12-07-44.png)

Scroll to the bottom of the page and click Save.

You will see this will not result in a new build. If you want to start a manual build you can do so by clicking `Start Build`. We will skip this for now and move on to the webhook part.

Click on `Configuration` tab.

Copy the GitHub Webook URL.

The webhook is in the structure

```text
https://c100-e.us-east.containers.cloud.ibm.com:31305/apis/build.openshift.io/v1/namespaces/example-health/buildconfigs/patientui/webhooks/<secret>/github
```

Make note of the secret. (Its the Alpha-Numeric string between `webhooks` and `github`) We will need it in the next step.

![Copy github webhook](https://dsc.cloud/quickshare/Shared-Image-2019-09-24-12-07-44.png)

> There is also the generic webhook url. This also works for Github. But the Github webhook captures some additional data from Github and is more specific. But if we were using some other git repo like Bitbucket or Gitlab we would use the generic one.

Back on our Github repo page go to `Setting > Webhooks`. Then click `Add Webhook`

![webhook page](https://dsc.cloud/quickshare/Shared-Image-2019-09-24-12-15-45.png)

In the Add Webhook page fill in the `Payload URL` with the url copied earlier from the build configuration. Change the `Content type` to `application/json` and fill in the secret.

Right now just the push event is being sent which is fine for our use.

Click on `Add webhook`

![add webhook](https://dsc.cloud/quickshare/Shared-Image-2019-09-24-12-17-53.png)

If the webhook is reachable by Github you will see a green check mark.

Back in our Openshift console we still would only see one build however. Because we added a webhook that sends us push events and we have no push event happening. Lets make one. The easiest way to do it is probably from the Github UI. Lets change some text in the login page.

Path to this file is `public/login.html` from the root of the directory. On Github you can edit any file by clicking the Pencil icon on the top right corner.

![edit page](https://dsc.cloud/quickshare/Shared-Image-2019-09-24-12-21-26.png)

Let's change the name our application to `Demo Health` (Line 21, Line 22). Feel free to make any other UI changes you feel like.

Once done go to the bottom and click `commit changes`.

Go to the openshift build page again. This happens quite fast so you might not see the running state. But the moment we made that commit a build was kicked off.

![Completed webhook build](https://dsc.cloud/quickshare/Shared-Image-2019-09-24-12-24-04.png)

In a moment it will show completed. Navigate to the overview page to find the route.

![route](https://dsc.cloud/quickshare/Shared-Image-2019-09-24-12-24-04.png)

> You could also go to `Applications > Routes` to find the route for the application.

If you go to your new route you will see your change.