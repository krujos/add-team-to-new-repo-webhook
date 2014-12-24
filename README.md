#Github web hook to add a team on repo create
At [Pivotal](http://pivotal.io) we use this web hook to add teams who are not owners but still need commit rights to our repos every time we create one. This code runs in [Cloud Foundry](http://run.pivotal.io) and uses environment variables to source the teams to add when the web hook is called. 

This is a generic app that should serve the same need for any github org.


##Environment variables

```GITHUB_SECRET``` is the secret you expect to be sent with the web hook. If the secret does not match ``X-Hub-Signature`` header in the incoming hook the request is dropped. 

```GITHUB_ACCESS_TOKEN``` is the personal access token to use for accessing the repo to add a collaborator. You can create a new token [here](https://github.com/settings/tokens/new). My access token has the ```admin:org``` permission. 

```GITHUB_COLLABORATOR``` is the id of the team you want to add to the new repo. You can obtain that by running ```curl https://<access_token>@api.github.com/orgs/<your_org>/teams``` and looking for the ```id``` attribute of the team you're interested in. 

```APP_NAME``` is the application name to use when pushing to Cloud Foundry. 

```APP_ROUTE``` is the subdomain to name when pushing, e.g. ``<whatever>.cfapps.io``. If this variable is not set it defaults to the app name

##Manifest generator
There's a manifest generator to help getting the environment variables correct, and avoid hardcoding them somewhere as checking in a manifest with your access key is probably a bad idea. 

```
➜  add-team-to-new-repo-webhook git:(master) ✗ export GITHUB_SECRET=secret 
➜  add-team-to-new-repo-webhook git:(master) ✗ export GITHUB_ACCESS_TOKEN=token
➜  add-team-to-new-repo-webhook git:(master) ✗ export GITHUB_COLLABORATOR=collab
➜  add-team-to-new-repo-webhook git:(master) ✗ export APP_NAME=ps-at-e-webhook
➜  add-team-to-new-repo-webhook git:(master) ✗ export APP_ROUTE=webhook
➜  add-team-to-new-repo-webhook git:(master) ✗ ./generate_manifest.sh
➜  add-team-to-new-repo-webhook git:(master) ✗ cat manifest.yml
applications:
- name: ps-at-e-webhook
  memory: 128M
  host: webhook

  env:
     GITHUB_SECRET: secret
     GITHUB_ACCESS_TOKEN: token
     GITHUB_COLLABORATOR: collab
```