# gitops-repo

GitOps state repository for the microservice deployment platform.

This repo is intentionally separate from:

- `/Users/macbookpro/Desktop/ISTAD/platform-infra-micro`

That infra repo keeps:

- Jenkins pipelines
- reusable Helm chart
- Dockerfile templates
- cluster/bootstrap helpers

This GitOps repo should keep only deployment state:

- generated Argo CD applications
- generated Helm values
- tenant namespace manifests

## Expected structure

```text
apps/
  <workspace-id>/
    <user-id>/
      namespace.yaml
      <project-name>/
        application.yaml
        values.yaml
```

Example:

```text
apps/demo-workspace/demo-user/
  namespace.yaml
  api-gateway/
    application.yaml
    values.yaml
  order-service/
    application.yaml
    values.yaml
```

## Automatic application creation

If you want Argo CD to create tenant applications automatically after Jenkins writes them here, apply the root app-of-apps manifest in:

- [root-app-of-apps.yaml](/Users/macbookpro/Desktop/ISTAD/gitops-repo/bootstrap/argocd/root-app-of-apps.yaml)

That root application watches the `apps/` folder recursively. When the deploy pipeline writes:

- `namespace.yaml`
- `application.yaml`
- `values.yaml`

Argo CD will discover the new child `Application` manifest and create it automatically on the next sync.

You only need to apply the root app once after replacing the placeholder `repoURL`.

## How it works

The Jenkins pipeline from the infra repo writes files here through:

- `/Users/macbookpro/Desktop/ISTAD/platform-infra-micro/jenkins/scripts/update-gitops.sh`

That script now:

- creates `namespace.yaml`
- creates `application.yaml`
- creates `values.yaml`
- references the Helm chart from the infra repo instead of copying the chart here

## What should not live here

Avoid putting these here:

- Jenkinsfiles
- shared Helm chart source
- framework Dockerfile templates
- build scripts

This repo should stay focused on desired cluster state.
