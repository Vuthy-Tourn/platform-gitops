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
- generated Helm values / Kustomize wrappers
- tenant namespace manifests
- shared reusable chart templates used by service-level Kustomize apps

## Expected structure

```text
apps/
  <workspace-id>/
    <user-id>/
      namespace.yaml
      <project-name>/
        application.yaml
        kustomization.yaml
        values.yaml
templates/
  charts/
    app-template/
      Chart.yaml
      values.yaml
      templates/
```

Example:

```text
apps/demo-workspace/demo-user/
  namespace.yaml
  api-gateway/
    application.yaml
    kustomization.yaml
    values.yaml
  order-service/
    application.yaml
    kustomization.yaml
    values.yaml
templates/charts/
  app-template/
    Chart.yaml
    values.yaml
    templates/
```

## Automatic application creation

If you want Argo CD to create tenant applications automatically after Jenkins writes them here, apply the root app-of-apps manifest in:

- [root-app-of-apps.yaml](/Users/macbookpro/Desktop/ISTAD/gitops-repo/bootstrap/argocd/root-app-of-apps.yaml)

That root application watches the `apps/` folder recursively. When the deploy pipeline writes:

- `namespace.yaml`
- `application.yaml`
- `kustomization.yaml`
- `values.yaml`

Argo CD will discover the new child `Application` manifest and create it automatically on the next sync.

You only need to apply the root app once after replacing the placeholder `repoURL`.

## How it works

The Jenkins pipeline from the infra repo writes files here through:

- `/Users/macbookpro/Desktop/ISTAD/platform-infra-micro/jenkins/scripts/update-gitops.sh`

That script now:

- creates `namespace.yaml`
- creates `application.yaml`
- creates `kustomization.yaml`
- creates `values.yaml`
- expects a shared reusable chart at `templates/charts/app-template`

## What should not live here

Avoid putting these here:

- Jenkinsfiles
- framework Dockerfile templates
- build scripts

This repo should stay focused on desired cluster state.
