# deployWithHelm
GitHub composite action to update tag in Helm infra repository, update K8s deployments with Helm and notify with MS Teams

## Assumptions ##
This composite action assumes the following:
1. That you use Helm for deploying your K8s cluster
2. That your Helm charts are stored in a dedicated GitHub repository
3. That each pod appears in your Helm chart along with its tag
4. That you wish to update the tag of pod-to-be-upgraded pod in your Helm repository after successful docker update of the relevant docker image 
5. That you use MS Teams, and wants to notify your team about deployment status
6. That your deployment workflow consists of the followings:
    1. Preliminary step: Update and push the relevant docker image and its tag using a different Job. For example, using [Pangea's buildUpdatePushGhcr GH Action](https://github.com/pangea-it/buildUpdatePushGhcr). Then in your deployment job you:
    2. Checkout your Helm repository to a dedicated location on the GH action runner
    3. Update the tag number of the deployed pod on the Helm repository with its updated version number, retrieved from the previous build job
    4. Create `kubeconfig` in `$HOME/.kube/config` of the runner
    5. Setup Helm
    6. Retrieve the Helm release name to be upgraded, by its namespace
    7. Perform `helm upgrade`
    8. Notify dedicated channel in MS Teams about the deployment status.

## Usage ##
```yaml
name: Build and deploy docker image with Helm
on:
  push:
    branches:
      - 'main' 
jobs:
  buildAndPushDocker:
    < some build and push to GHCR job, e.g. https://github.com/pangea-it/buildUpdatePushGhcr >
    ...
    ...
    
  deployment:
    needs: buildAndPushDocker
    runs-on: ubuntu-latest
```
    
## Parameters ##
Name                  | Required                                      
-------------         | -------------                                
token                 | Yes. Defaults to `GITHUB_TOKEN`.
helm_repo             | Yes. GitHub repository with your Helm charts
helm_folder           | Yes. Folder path, within your Helm charts repo, within which you'd execute the `helm upgrade` command. Note: By default, Helm charts repository will be installed in `infra` sub-directory in the runner. So, for example, if the folder within which you'd execute the `helm upgrade` command is in `some-subfolder` which is located on the repository's root folder, you should input `some-subfolder` here.
tag_regex             | Yes. Defaults to: `(${{github.event.repository.name}}[\r\n]+\s*tag:\s*)(\d*\.\d*\.\d*)`. Regex groups are supported.
tag_replace_regex     | Yes. Defaults to: `"$1${{needs.buildAndPushDocker.outputs.tagversion}}"`. Regex groups can be referred with `$<group no>`
tag_replace_file      | Yes. Defaults to `^infra/${{ inputs.helm_folder }}/values.yaml`. Path to the Helm file where the tag needs to be updated, in Regex format. By default, Helm charts repository will be installed in `infra` sub-directory in the runner. So for example, to update `infra/special-subfolder/values.yaml`, use: `^infra/special-subfolder/values.yaml`
kube_config           | Yes. Secret name that contains the `kubeconfig` contents
k8s_namespace         | Yes. Namespace name of your resources.
use_teams             | Yes. Defaults to `true`. Indicates whether deployment status notification will be sent to dedicated MS Teams channel.
teams_webhook         | No. Required if `use_teams` is set to `true`. Secret name with MS Teams webhook

