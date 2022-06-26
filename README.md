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
    1. Preliminary step: Update and push the relevant docker image and its tag using a different Job, named, by default, `buildAndPushDocker`. For example, using [Pangea's buildUpdatePushGhcr GH Action](https://github.com/pangea-it/buildUpdatePushGhcr). Then in your deployment job you:
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
    steps:
      - name: deployWithHelm
        uses: pangea-it/deployWithHelm@main
        with:
          token: ${{secrets.PAT}}
          helm_repo: some-organization/helm-infra-repo
          helm_folder: special-folder
          tag_replace_regex: "$1${{needs.buildAndPushDocker.outputs.tagversion}}"
          tag_replace_file: ^infra/special-folder/values.yaml
          tag_number: ${{needs.buildAndPushDocker.outputs.tagversion}}
          kube_config: ${{ secrets.KUBECONFIG }}
          k8s_namespace: my-namespace
          teams_webhook: ${{ secrets.TEAMS_WEBHOOK }}
```
    
## Parameters ##
Name                  | Required                                      
-------------         | -------------                                
token                 | Yes. Defaults to `GITHUB_TOKEN`
helm_repo             | Yes. GitHub repository with your Helm charts
helm_folder           | Yes. Folder path, within your Helm charts repo, within which you'd execute the `helm upgrade` command. Note: By default, Helm charts repository will be installed in `infra` sub-directory in the runner. So, for example, if the folder within which you'd execute the `helm upgrade` command is in `some-subfolder` which is located on the repository's root folder, you should input `some-subfolder` here
tag_regex             | Yes. Defaults to: `(${{github.event.repository.name}}[\r\n]+\s*tag:\s*)(\d*\.\d*\.\d*)`. Previously-updated Docker image tag to be updated in the Helm repo. Regex groups are supported
tag_replace_regex     | Yes. Regex groups can be referred with `$<group no>`. Regex pattern to replace the tag of the relevant docker image in the Helm repository.For example: `"$1${{needs.buildAndPushDocker.outputs.tagversion}}"`.
tag_replace_file      | Yes. Path to the Helm file where the tag needs to be updated, in Regex format. By default, Helm charts repository will be installed in `infra` sub-directory in the runner. So for example, to update `infra/special-subfolder/values.yaml`, use: `^infra/special-subfolder/values.yaml`
tag_number            | Yes. Updated docker image tag number from previous build-and-deploy job. If [Pangea's buildUpdatePushGhcr GH Action](https://github.com/pangea-it/buildUpdatePushGhcr) used to build and push docker image, set to: `${{needs.<build and push job name, e.g. buildAndPushDocker>.outputs.tagversion}}`
kube_config           | Yes. Secret name that contains the `kubeconfig` contents
k8s_namespace         | Yes. Namespace name of your resources
use_teams             | Yes. Defaults to `true`. Indicates whether deployment status notification will be sent to dedicated MS Teams channel
teams_webhook         | No. Required if `use_teams` is set to `true`. Secret name with MS Teams webhook

## Credits ##
The following GitHub Actions are used in this action:
+ actions/checkout
+ mingjun97/file-regex-replace
+ azure/setup-helm
+ skitionek/notify-microsoft-teams
