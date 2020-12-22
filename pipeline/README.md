# Sample gitops pipeline

Very simple pipeline, that takes manifests from the source git repo and commits to the gitops repo to the `test` and `qa` branches. And creates pull request for the `production` branch.

Files:

- argocd-trader-pipeline.yaml - pipeline definition
