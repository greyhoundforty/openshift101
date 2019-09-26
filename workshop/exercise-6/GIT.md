## Add Github Webhook integration
So far we have been doing alot of manual deployment. In cloud-native world we want to move away from manual work and move toward automation. Wouldn't it be nice if our application rebuilt on git push events? Git webhooks are the way its done and openshift comes bundled in with git webhooks. Let's set it up for our project.

To be able to setup git webhook we need to have elevated permission to the project so we'll be using our previously forked repository `https://github.com/<username>/node-s2i-openshift`. 

From our openshift dashboard for our project. Select `Builds > Builds`

![Goto Build](https://dsc.cloud/quickshare/Shared-Image-2019-09-25-17-05-36.png)

Select the patientui build. As of now this should be the only build on screen.

![Select Build](https://dsc.cloud/quickshare/Shared-Image-2019-09-25-17-06-32.png)

Click on `Action` on the right and then select `Edit`

![Edit Build](https://dsc.cloud/quickshare/Shared-Image-2019-09-25-17-07-43.png)

Change the `Git Repository URL` to our forked repository.

![New Repo](https://dsc.cloud/quickshare/Shared-Image-2019-09-25-17-09-46.png)

Scroll to the bottom of the page and click Save.

You will see this will not result in a new build. If you want to start a manual build you can do so by clicking `Start Build`. We will skip this for now and move on to the webhook part.

Click on `Configuration` tab.

Copy the GitHub Webook URL.

The webhook is in the structure

```text
https://c100-e.us-east.containers.cloud.ibm.com:31305/apis/build.openshift.io/v1/namespaces/example-health/buildconfigs/patientui/webhooks/<secret>/github
```

Make note of the secret. (Its the Alpha-Numeric string between `webhooks` and `github`) We will need it in the next step.

![Copy github webhook](https://dsc.cloud/quickshare/Shared-Image-2019-09-25-17-11-16.png)

> There is also the generic webhook url. This also works for Github. But the Github webhook captures some additional data from Github and is more specific. But if we were using some other git repo like Bitbucket or Gitlab we would use the generic one.

Back on our Github repo page go to `Setting > Webhooks`. Then click `Add Webhook`

![webhook page](https://dsc.cloud/quickshare/Shared-Image-2019-09-25-17-14-08.png)

In the Add Webhook page fill in the `Payload URL` with the url copied earlier from the build configuration. Change the `Content type` to `application/json` and fill in the secret.

Right now just the push event is being sent which is fine for our use.

Click on `Add webhook`

![add webhook](https://dsc.cloud/quickshare/Shared-Image-2019-09-25-17-16-30.png)

If the webhook is reachable by Github you will see a green check mark.

Back in our Openshift console we still would only see one build however. Because we added a webhook that sends us push events and we have no push event happening. Lets make one. The easiest way to do it is probably from the Github UI. Lets change some text in the login page.

Path to this file is `site/public/login.html` from the root of the directory. On Github you can edit any file by clicking the Pencil icon on the top right corner.

![edit page](https://dsc.cloud/quickshare/Shared-Image-2019-09-25-17-18-55.png)

Let's change the name our application to `Demo Health` (Line 21, Line 22). Feel free to make any other UI changes you feel like.

Once done go to the bottom and click `commit changes`.

Go to the openshift build page again. This happens quite fast so you might not see the running state. But the moment we made that commit a build was kicked off.

![Completed webhook build](https://dsc.cloud/quickshare/running-build.png)

In a moment it will show completed. Navigate to the overview page to find the route.

![route](https://dsc.cloud/quickshare/Shared-Image-2019-09-25-17-23-31.png)

> You could also go to `Applications > Routes` to find the route for the application.

If you go to your new route you will see your change.

![Updated deployment](https://dsc.cloud/quickshare/Shared-Image-2019-09-25-17-24-00.png)


