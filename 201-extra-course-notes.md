# OpenShift 201

Course site: https://ocp201-content.pathfinder.gov.bc.ca/ 
Lab site: https://ocp201-labs.pathfinder.gov.bc.ca/

## Flows
GitHub Flow is a more simple flow than Git Flow. Git Flow may be too much for small teams; GitHub Flow may be too simple for larger teams. 
Pathfinder recommends GitHub Flow
Commit often; merge from master often
The `master` branch should ALWAYS be deployable

## GitOps
Kubernetes namespace = OpenShift project

## Templates & Labels
- see OpenShift Templates Styleguide in https://github.com/BCDevOps/platform-services/blob/master/CONTRIBUTING.md 
- can have a parameter file for each environment
- resist the urge to turn everything into a parameter. Only parameterize the variables that may be change. A parameter for everything will be overwhelming
- OpenShift templates ONLY work in OpenShift (not in Kubernetes)
- within your own OpenShift namespace, the command `oc get templates` lists all the templates within your namespace. To view templates within the OpenShift namespace, use command `oc get templates -n openshift` (this returns different results from the first command). Can view the details of a template using `oc describe template <template-name> -n openshift`
- a template can be exported to your local env so that the yaml structure can be viewed
- `oc apply -f <template-name>.yaml` from local env will then apply the template to your namespace (whichever namespace you're logged into from the CLI)
- can then deploy the template using `oc process <template-name> <specify parameters>`

## Helm
- package manager for Kubernetes (and also a deployment manager)
- on Mac, can be installed using `brew install kubernetes-helm` <-- easiest way to install. 
- `helm init --client-only` prevents Tiller from being installed. Nobody likes Tiller so it's being deprecated & replaced.
- Helm Templates lab exercise: see `andrea-williams-prometheus.yaml` and `loki_template.yaml`

## Prometheus
- https://github.com/helm/charts/tree/master/stable/prometheus
- used for monitoring, reporting metrics

## OpenShift Deployment Artifact for Grafana (lab exercise)
- started with `grafana_template.yaml`. Deleted a bunch of objects that caused things to break: the new file is `grafana_template_revised.yaml`.
- if running `oc apply -f grafana_template.yaml` returns any errors, all objects have to be deleted before the template can be applied again (otherwise errors will be returned even if the revised template is correct). To delete all objects, run `oc delete all,configmap -l app=<app-name>`
- you know that the template is correct if you run the `oc apply` command for the template multiple times. First time the template is applied, there will be no errors returned. Subsequent times the template is applied (without deleting any of the objects), CLI will output `<object-name> unchanged`
> my deploymentconfig section might not be correct in the revised grafana template. CLI output reports that the deploymentconfig for openshift is "configured" (instead of unchanged) every time oc apply is run

- created OpenShift template for Grafana: `openshift_grafana_template.yaml`. Deployed with command `oc process grafana-template -p GRAFANA_SERVICE_NAME=second-grafana-app -p LOKI_SERVICE_NAME=andrea-williams-loki -p PROMETHEUS_SERVICE_NAME=andrea-williams-prometheus-andrea-williams-prometheus -p ROUTE_SUBDOMAIN=pathfinder.gov.bc.ca -p NAMESPACE=4zq6uj-andrea-williams-ocp201-tst-dev | oc apply -f -`. The new grafana app (second-grafana-app) should appear in OpenShift dashboard
- created OpenShift template for Loki: `openshift_loki_template.yaml`. Deployed with command `oc process -f openshift_loki_template.yaml -p LOKI_SERVICE_NAME=loki-2 -p NAMESPACE=4zq6uj-andrea-williams-ocp201-tst-dev -p PROMETHEUS_SERVICE_NAME=andrea-williams-prometheus-andrea-williams-prometheus | oc apply -f -`. It works.
- also see OpenShift template for Prometheus: `openshift_prometheus_template.yaml`

## Jenkins
- when using persistent Jenkins, you (as a team) are responsible for maintaining your Jenkins instance - i.e., you have to manually perform upgrades, maintain plugins, etc. Not the case with ephermeral instances of Jenkins
- Jenkins should only be run in tools namespace
- **Blue Ocean**: updated UI for Jenkins
- code in "Pipeline Sample Config" slide is out of date
- "Jenkins agent" is the new slave
- recommended simplest way to enable build/pipeline notifications is to install plugin in Jenkins. Can send notifications to Rocketchat or email
- in ideal world, the only "user" that should have write-access to prod environment is Jenkins. Devs should only have read-access to prod

> Major Gotcha for all the Jenkins exercises: if anything in Jenkins fails, the deployment/statefulSet/whatever needs to be deleted in OpenShift, otherwise Jenkins will keep failing even if everything else has been configured correctly

### Jenkins Build Pipeline lab exercise
- in OpenShift web console, have to modify membership in -tools namespace so that the default service account for the -dev namespace can pull images. Access can be set to `system:image-puller`