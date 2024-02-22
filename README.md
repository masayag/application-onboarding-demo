# Application onboarding
This project demonstrate how a workflow can interact with an external service, jira, and send notifications to backstage

## Prerequisites
1. Have a running JIRA instance (ie: free cloud instance)
2. Have orchestrator installed


## Usage
* Review the [application.properties](src/main/resources/application.properties) file and set the values accordingly.
* Create a [webhook](https://redhat-gfarache.atlassian.net/plugins/servlet/webhooks) in your JIRA for ISSUE events that point to <BACKSTAGE_URL>/api/orchestrator/webhook/jira. 
    * Please note that this endpoint has been created for demo purposes so it is not generic enough to support multiple use cases. This step can be skipped if you are willing to send the cloud event yourself.
* Be sure to include the Notification plugin in your Backstage instance.
* Be sure to include the Orchestrator plugins on at least the versions: 
    * `1.5.2` for `@janus-idp/backstage-plugin-orchestrator`
    * `1.4.1` for `@janus-idp/backstage-plugin-orchestrator-backend-dynamic`


## Generate image and manifest
To generate the image, run:
```bash
REGISTRY_REPO=<your org> WORKFLOW_ID=application-onboarding make prepare-workdir build-image push-image 
```
To generate the manifest, run:
```bash
cd application-onboarding
curl -L https://github.com/rgolangh/kie-tools/releases/download/0.0.2/kn-workflow-linux-amd64 -o kn-workflow && chmod +x kn-workflow
./../kn-workflow gen-manifest --namespace ""
```

Then make sure the manifests are prod-ready:
```bash
yq -iy 'del(.metadata.annotations."sonataflow.org/profile")' manifests/01-sonataflow*.yaml
```
And set the manifest's image to the generated image:
```bash
yq -iy '.spec.podTemplate.container.image="quay.io/gfarache/serverless-workflow-application-onboarding:latest"' manifests/01-sonataflow*.yaml
```

## Deploy

Now you can simply deploy all the files in the `manifests` folder:
```bash
$ kubectl apply -f manifests -n <your NS>
configmap/01-application-onboarding-resources created
sonataflow.sonataflow.org/application-onboarding created
configmap/application-onboarding-props created
```
