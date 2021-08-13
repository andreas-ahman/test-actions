# dpci-gke-infrastructure

Runs infrastructure pipeline for gke clusters

## Git Flow

There are 3 branches:

* production - contains terraform files that's running in dev/stage/prod
* develop - contains terraform files that's running in test
* workflows - contains all Github Workflow files

All development should be done in feature branches based on the `develop` branch.
The feature is then merged to the develop branch and then finally to production.

### Deploying changes

Create a PR for the env you intend to deploy.

For managing deployments we're using the [Slash Command Dispatch Action](https://github.com/peter-evans/slash-command-dispatch)

To deploy changes simply comment the PR with like this:

* /deploy ccoe-europe-west4-test-1

## Terraform

### Directory structure

The directory structure is composed as follows. A terraform root directory, a subdirectory for each cluster.

Every cluster must have a `${env}.tfvars` file.

```yaml
terraform/
        cluster1/
            test.tfvars
            dev.tfvars
            stage.tfvars
            prod.tfvars
            main.tf
        cluster2/
```

## Pipelines

There are currently 3 pipelines.

* `format.yaml` - runs on every push to validate terraform format.
* `deploy_command.yml` - Runs when a pull request is raised. Runs terraform plan
* `destroy.yaml`- Runs on repository_dispatch from dpci-gke-admin-apps repo. Runs terraform deploy on **TEST** cluster only.

## Setup pipeline

### Variables

The pipeline needs a few secrets set in github to work properly

* `CONFIG_PROJECT` - The full GCP project that contains the secrets (does not have to be the same as the environment to deploy to)
* `INGKA_SSHKEY` - The private key to be able to check out the terraform modules
* `SECRET_MANAGER_KEY` - Secrets manager vault manager key
* `SECRET_MANAGER_EMAIL` - Secrets manager vault manager email

The secret manager keys will be used to fetch the following information from the Google secrets manager,

#### Secrets from secret manager (config project)

* `tf-gke-config` - Service account (json) for the config access account, this account is used to be able to fetch secrets and to store the terraform state in the CONFIG_PROJECT
* `tf-gke-${ENVIRONMENT}` - Service account (base64+json) used for access and deployments to the respective environment. There will be one account per environment to deploy to.
`${ENVIRONMENT}` will be expanded in the pipeline to the environment that it's running in.