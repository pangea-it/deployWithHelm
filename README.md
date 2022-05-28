# deployWithHelm
GitHub composite action to update tag in Helm infra repository, update K8s deployments with Helm and notify with MS Teams

## Assumptions ##
This composite action assumes the following:
1. That you use Helm for deploying your K8s cluster
2. That your Helm charts are stored in a GitHub repository
3. That each pod appears in your Helm chart along with its tag
4. That you wish to update the tag of upgraded pod in your Helm repository after successful upgrade
5. That you use MS Teams, and wants to notify your team about deployment status
6. That your deployment job consists of the followings:
    1. Checkout your Helm repository
    2. Update the tag number of the deployed pod on the Helm repository with its updated version number, retrieved from previous job (see for example [Pangea's buildUpdatePushGhcr GH Action](https://github.com/pangea-it/buildUpdatePushGhcr))
    3. Create `kubeconfig` in `$HOME/.kube/config`
    4. Setup Helm
    5. Retrieve the Helm release name to be upgraded, by its namespace
    6. Perform `helm upgrade`
    7. Notify dedicated channel in MS Teams about the deployment status.

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
token                 | Yes. Defaults to `GITHUB_TOKEN`. Used for the NPM registry as well.             
use_snyk              | Yes. Defaults to `true`. Set to `false` to ignore Snyk.
perform_tests         | Yes. Defaults to `true`. Set to `false` to ignore unit tests.
snyk_token            | No. Required only if `use_snyk` is set to `true`.                                          
npm_registry          | Yes. Defaults to `https://npm.pkg.github.com/` 
npm_registry_scope    | Yes. Example: `'@example-org'`                                        
docker_login_registry | Yes. Defaults to `ghcr.io`                     
docker_login_user     | Yes. Defaults to `github.actor`                
ghcr_tag_owner        | Yes. Defaults to `github.repository_owner`     
docker_image_name     | Yes. Defaults to `github.event.repository.name`

