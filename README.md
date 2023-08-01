# helm-charts

## Releasing a change to your chart

### Packaging your change

You can use the included `./package-chart.sh` script which invokes
the `helm` CLI to package your change. You must have the `helm` CLI installed.

Don't forget to bump the version in `Chart.yaml`.

### Publishing your change

A Github Action automatically publishes new charts to Google Artifact Registry. The action runs after every push to `main` if `Chart.yaml` has been modified.

### Rotating Secrets

If you need to change the key that allows Github Actions to communicate with Artifact Registry (named GAR_KEY_BASE64):

1. Create and download a new JSON key in GCP
2. Encode it as Base64 and upload in same step: `cat /path/to/keyfile.json | jq -r 'tostring' | base64 | gh secret set GAR_KEY_BASE64 --repo UWIT-IAM/helm-charts --app actions`.
3. If you don't want to use the `gh` CLI, skip the last command and upload it to the repository manually.

### Debugging

#### Did the new release get picked up by automation?

Compare the `sha256sum` of `charts/index.yaml` to what the HelmRepository `helm-chart-repository` says:

```
$ kubectl get helmrepository helm-chart-repository
NAME                    URL                                                         AGE    READY   STATUS
helm-chart-repository   https://uw-iti-app-platform.github.io/helm-charts/charts/   321d   True    stored artifact for revision '1ae29d2f07b8a9ec52f8eb5ccc03acbf6ad86b58a9a388fd880cfe29edda1782'


$ sha256sum charts/index.yaml
1ae29d2f07b8a9ec52f8eb5ccc03acbf6ad86b58a9a388fd880cfe29edda1782  charts/index.yaml
```

If they don't match, take a look at the Github Action logs to see if anything is amiss.


#### Is the release valid?

Check an app that should be using that release via `kubectl get helmrelease YOUR_APP_NAME`

If you see an error there is no easy fix, but you can render the chart to see what happens and try and diagnose via `helm template basic-web-service/ | less`