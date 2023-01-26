### DefectDojo on Openshift via Helm, deployed with a Gitlab Pipeline

This repo contains the modified values-nonprod.yaml for the helm chart of DefectDojo: https://github.com/DefectDojo/django-DefectDojo/tree/master/helm/defectdojo
Any changes made for the productive environment can be stored in values-prod.md.

To start one needs to create a new Open Project. For this we used the oc tool. After signing into Openshift via the CLI, one can create a new project by:

 	`oc new-project <project-name>`
	## further commands?

The helm repo contains a setup for Prometheus.
To adapt these settings one needs to change the placeholders inside the values.yaml (e.g. <insert webhost address>).

DefectDojo is being deployed via a Gitlab CI/CD Pipeline, whenever a change in main occurs.

When deploying this repo for the first time via Gitlab, one has to modify the .gitlab-ci.yml and insert the project specific values (e.g. namespace), replacing the placeholders (e.g. <your-namespace>). In addition the following Secrets should be added in Settings -> CI/CD -> Variables
    -ADMIN_PASSWORD
    -KEYCLOAK_SECRET
    -POSTGRESQL_PASSWORD
    -OC_TOKEN

The OC Token has to first be generated, in order to connect your Gitlab CICD Pipeline to automatically deploy your latest versions on Openshift.
To achieve this, follow these steps:

1. Check if you have created your .gitlab-ci.yml file correctly and it runs through smoothly (by deploying manually).
2. Log in to your namespace via your oc CLI tool.

    `oc login --token=${OC_TOKEN} --server=<you server address>`

3. Run the following commands to create a service account:

    `oc create sa gitlab-ci -n <project-name>`

    `oc policy add-role-to-user edit    system:serviceaccount:<project-name>:gitlab-ci -n <project-name>`

4. Now create the token of said service account:

for example via vim, by typing `vim octoken.yaml` in the cli.

```
	apiversion: v1
	kind: Secret
	metadata:
	  name: secret-sa-sample
	  annotations:
	    kubernetes.io/service-account.name: "sa-name" ###get this value by typing: oc get sa into the CLI
	type: kubernetes.io/service-account-token
```

5. Run command:

    `oc create -f ocToken.yaml`

6. Run the following command to display the Token:

    `oc --namespace <project-name> get secrets secret-sa-sample -o jsonpath='{ .data.token }' | base64 -d`

7. Copy the displayed token into the formally created OC_TOKEN Variable.
