# Sample gitops pipeline

Very simple pipeline, that takes manifests from the source git repo and commits to the gitops repo to the `test` and `qa` branches. And creates pull request for the `production` branch.

Files:

- argocd-demo-pipeline.yaml - pipeline definition
- ibm-setup-v2-1-26.yaml - setup task definition
- ibm-gitops-v2-1-26.yaml - gitops task definition
- ibm-git-pullrequest-v0-0-1 - pull request task definition
- gitops-repo-cm.yaml - config map with gitops repository params
- git-credentials-secret.yaml - secret with username and token for gitops repo
- agrocd-demo-pipeline-listener.yaml - listener definition
- agrocd-demo-pipeline-template.yaml - trigger template
- agrocd-demo-pipeline-binding.yaml - trigger binding
