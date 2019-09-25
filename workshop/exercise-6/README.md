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

## Deploying the Example Health App from CLI

1. Get the source code for the `Example Health` app

	* Fork the repository `https://github.com/IBM/node-s2i-openshift` to your own organization,

	* Clone the forked repo in your own organization to your localhost,

		```console
		$ git clone https://github.com/<username>/node-s2i-openshift
		Cloning into 'node-s2i-openshift'...
		remote: Enumerating objects: 94, done.
		remote: Counting objects: 100% (94/94), done.
		remote: Compressing objects: 100% (82/82), done.
		remote: Total 509 (delta 27), reused 54 (delta 12), pack-reused 415
		Receiving objects: 100% (509/509), 7.27 MiB | 3.12 MiB/s, done.
		Resolving deltas: 100% (276/276), done.
		```

	* Run the `Health Example` app on your localhost to make sure it's running correctly,

		```console
		$ cd node-s2i-openshift
		$ cd site
		$ npm install
		$ npm start
		$ open http://localhost:8080/
		```

2. Run the Example Health app with Docker,

	* In the directory `./site` create a new file `Dockerfile`,

		```console
		$ touch Dockerfile
		```

	* Edit the `Dockerfile` and add the following commands,

		```text
		FROM node:10-slim

		USER node

		RUN mkdir -p /home/node/app
		WORKDIR /home/node/app

		COPY --chown=node package*.json ./
		RUN npm install
		COPY --chown=node . .

		ENV HOST=0.0.0.0 PORT=3000

		EXPOSE ${PORT}
		CMD [ "node", "app.js" ]
		```

	* Run the app with Docker,

		```console
		$ docker stop example-health
		$ docker rm example-health
		$ docker build --no-cache -t example-health .
		$ docker run -d --restart always --name example-health -p 3000:3000 example-health
		da106f3b5a06a00ea8bf56c54e29f6e38405a77c6dec3e461e3062aa823d8a4f
		```

3. Build and Push the Image to your public Docker Hub Registry.

	* Make sure to change the <username> by the username of your Docker Hub account,

	```bash
	$ docker build --no-cache -t example-health .
	Sending build context to Docker daemon  8.229MB
	Step 1/10 : FROM node:10-slim
	 ---> 8d33f30db9b5
	... and more
	Successfully built aaf90ce81dd7
	Successfully tagged example-health:latest

	$ docker tag example-health:latest <username>/example-health:1.0.0
	$ docker login -u <username>
	Password:
	Login Succeeded

	$ docker push <username>/example-health:1.0.0
	The push refers to repository [docker.io/<username>/example-health]
	b33f2248b6f9: Pushed
	195f723f9ebb: Pushed
	0912774a40f4: Pushed
	3558c6f90d27: Pushed
	4d1d690b5181: Mounted from <username>/example-health
	bc272904b2c4: Mounted from <username>/example-health
	784c13bc7926: Mounted from <username>/example-health
	0e0d79e2c080: Mounted from <username>/example-health
	e9dc98463cd6: Mounted from <username>/example-health
	1.0.0: digest: sha256:a329778ce422e3d25ac9ff70b5131a9de26184a1e94b6d08844ea4f361519fd7 size: 2205
	```

1. Login to the Remote OpenShift Cluster

	* Login to the OpenShift cluster web console,
	* From the logged in user drop down in the top right of the web console, select `Copy Login Command`,
	* The login command will be copied to the clipboard,
	* In your terminal, paste the login command, e.g.

		```console
		$ oc login https://c100-e.us-south.containers.cloud.ibm.com:30403 --token=jWX7a04tRgpdhW_iofWuHqb_Ygp8fFsUkRjOK7_QyFQ
		```

4. Create a new Project

	* Create a new project `example-health-ns`,

		```console
		$ oc new-project example-health-ns
		Now using project "example-health-ns" on server "https://c100-e.us-south.containers.cloud.ibm.com:30403".

		You can add applications to this project with the 'new-app' command. For example, try:

			oc new-app centos/ruby-25-centos7~https://github.com/sclorg/ruby-ex.git

		to build a new example application in Ruby.
		```

	* Use the `example-health-ns` project,

		```console
		$ oc project example-health-ns
		Already on project "example-health-ns" on server "https://c100-e.us-south.containers.cloud.ibm.com:30403".
		$ oc project
		Using project "example-health-ns" on server "https://c100-e.us-south.containers.cloud.ibm.com:30403".
		```

5. Deploy the `Example Health` app using the Docker image,

	* Create the application, and replace <username> by the username of your Docker Hub account,

		```console
		$ oc new-app <username>/example-health:1.0.0
		--> Found Docker image aaf90ce (8 minutes old) from Docker Hub for "<username>/example-health:1.0.0"

			* An image stream tag will be created as "example-health:1.0.0" that will track this image
			* This image will be deployed in deployment config "example-health"
			* Port 3000/tcp will be load balanced by service "example-health"
			* Other containers can access this service through the hostname "example-health"

		--> Creating resources ...
			imagestream.image.openshift.io "example-health" created
			deploymentconfig.apps.openshift.io "example-health" created
			service "example-health" created
		--> Success
			Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
			'oc expose svc/example-health'
			Run 'oc status' to view your app.
		```

	* This will create an ImageStream, Deployment, a Pod, and a Service resource for the `Example-Health` app,

6. Expose the `Example-Health` service,

	* The last thing to do is to create a route. By default, services on OpenShift are not publically available. A route will expose the service publically to external traffic.

		```console
		$ oc expose svc/example-health
		route.route.openshift.io/example-health exposed
		```

	* View the status,

		```console
		$ oc status
		In project example-health-ns on server https://c100-e.us-south.containers.cloud.ibm.com:30403

		http://example-health-example-health-ns.cda-openshift-cluster-1c0e8bfb1c68214cf875a9ca7dd1e060-0001.us-south.containers.appdomain.cloud to pod port 3000-tcp (svc/example-health)
		dc/example-health deploys istag/example-health:1.0.0
			deployment #1 deployed 5 minutes ago - 1 pod

		2 infos identified, use 'oc status --suggest' to see details.
		```

7.  Review the `Example-Health` app in the web console,

	* Go to `My Projects` via URI `/console/projects`,

		![My Projects](https://dsc.cloud/quickshare/Shared-Image-2019-09-25-15-38-01.png)

	* Select the project `example-health-ns`, unfold the `DEPLOYMENT CONFIG` for `example-health` application details,

		![Example Health details](https://dsc.cloud/quickshare/Shared-Image-2019-09-25-15-39-07.png)

	* In the `NETWORKING` section, click the `Routes - External Traffic` link, e.g. http://example-health-example-health-ns.roks-rt-007632d2d5235aabd90e420d6faed9fd-0001.us-south.containers.appdomain.cloud/login.html

	* This opens the Example Health app in a new tab of your browser,
	* Login with `test:test`,

		![Example Health details](https://dsc.cloud/quickshare/Shared-Image-2019-09-25-15-40-10.png)

### Add Health Checks 
Just like in our main lab, we'll add some Health checks to our application using the `probe` command. A probe is a Kubernetes action that periodically performs diagnostics on a running container. 

```
$ oc set probe dc/example-health --readiness --get-url=http://:3000/info --initial-delay-seconds=5 --timeout-seconds=2
deploymentconfig.apps.openshift.io/v3 probes updated

$ oc set probe dc/example-health --liveness --get-url=http://:3000/info --initial-delay-seconds=5 --timeout-seconds=2
deploymentconfig.apps.openshift.io/example-health probes updated

$ oc status
In project example-health-ns on server https://c100-e.us-south.containers.cloud.ibm.com:32284

http://example-health-example-health-ns.roks-rt-007632d2d5235aabd90e420d6faed9fd-0001.us-south.containers.appdomain.cloud to pod port 3000-tcp (svc/example-health)
  dc/example-health deploys istag/example-health:latest
    deployment #3 running for 33 seconds - 1 pod
    deployment #2 deployed about a minute ago
    deployment #1 deployed 7 minutes ago
```

### Add Resource Limits 

```
$ oc set resources DeploymentConfigs/example-health --limits=cpu=30m,memory=100Mi --requests=cpu=3m,memory=40Mi
deploymentconfig.apps.openshift.io/example-health resource requirements updated
```

### Add Autoscaler 
You can create a horizontal pod autoscaler with the oc autoscale command and specify the minimum and maximum number of pods you want to run, as well as the CPU utilization or memory utilization your pods should target.

```
$ oc autoscale dc/example-health --min=1 --max=5 --cpu-percent=20
horizontalpodautoscaler.autoscaling/v3 autoscaled
```

## Add Github Webhook integration
So far we have been doing alot of manual deployment. In cloud-native world we want to move away from manual work and move toward automation. Wouldn't it be nice if our application rebuilt on git push events? Git webhooks are the way its done and openshift comes bundled in with git webhooks. Let's set it up for our project.

To be able to setup git webhook we need to have elevated permission to the project so we'll be using our previously forked repository. The repo we have been using so far we don't own it. But since its opensource we can easily fork it and make it our own.

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