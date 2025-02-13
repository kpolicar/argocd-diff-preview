<p align="center">
  <img src="./docs/assets/logo.png" width="70%" />
</p>

Argo CD Diff Preview is a tool that renders the diff between two branches in a Git repository. It is designed to render manifests generated by Argo CD, providing a clear and concise view of the changes between two branches. It operates similarly to Atlantis for Terraform, creating a plan that outlines the proposed changes.

### 3 Example Pull Requests:
- [Helm Example | Internal Chart](https://github.com/dag-andersen/argocd-diff-preview/pull/16)
- [Helm example | External Chart: Nginx](https://github.com/dag-andersen/argocd-diff-preview/pull/15)
- [Kustomize Example](https://github.com/dag-andersen/argocd-diff-preview/pull/12)

![](./docs/assets/example-1.png)


## Why do we need this?

In the Kubernetes world, we often use templating tools like Kustomize and Helm to generate our Kubernetes manifests. These tools make maintaining and streamlining configuration easier across applications and environments. However, they also make it harder to visualize the application's actual configuration in the cluster.

Mentally parsing Helm templates and Kustomize patches is hard without rendering the actual output. Thus, making mistakes while modifying an application's configuration is relatively easy.

In the field of GitOps and infrastructure as code, all configurations are checked into Git and modified through PRs. The code changes in the PR are reviewed by a human, who needs to understand the changes made to the configuration. This is hard when the configuration is generated through templating tools like Kustomize and Helm.

## Overview

![](./docs/assets/flow_dark.png)

The safest way to make changes to you Helm Charts and Kustomize Overlays in your GitOps repository is to let Argo CD render them for you. This can be done by spinning up an ephemeral cluster in your automated pipelines. Since the diff is rendered by Argo CD itself, it is as accurate as possible.

## Features

- Renders manifests generated by Argo CD
- Provides a clear and concise view of the changes
- Does not require access to your real cluster or Argo CD instance. The tool runs in complete isolation.
- Can be run locally before you open the pull request
- Supports private repositories and Helm charts
- Supports multi-source applications
- Render resources from external sources (e.g., Helm charts). For example, when you update the Helm Chart version of `nginx`, you can see what exactly changed. [PR example](https://github.com/dag-andersen/argocd-diff-preview/pull/15) 

---

> [!TIP]
> 
> ## Try demo locally with 3 simple commands!
> 
> First, make sure Docker is running. E.g., run `docker ps` to see if it's running.
> 
> Second, run the following 3 commands:
> 
> ```bash
> git clone https://github.com/dag-andersen/argocd-diff-preview base-branch --depth 1 -q 
> git clone https://github.com/dag-andersen/argocd-diff-preview target-branch --depth 1 -q -b helm-example-3
> docker run \
>    --network host \
>    -v /var/run/docker.sock:/var/run/docker.sock \
>    -v $(pwd)/output:/output \
>    -v $(pwd)/base-branch:/base-branch \
>    -v $(pwd)/target-branch:/target-branch \
>    -e TARGET_BRANCH=helm-example-3 \
>    -e REPO=dag-andersen/argocd-diff-preview \
>    dagandersen/argocd-diff-preview:v0.0.33
> ```
> 
> and the output would be something like this:
> 
> ```
> ...
> 🚀 Creating cluster...
> 🦑 Installing Argo CD...
> ...
> 🌚 Getting resources for base-branch
> 🌚 Getting resources for target-branch
> ...
> 🔮 Generating diff between main and helm-example-3
> 🙏 Please check the ./output/diff.md file for differences
> ```
> 
> Finally, you can view the diff by running `cat ./output/diff.md`. The diff should look something like [this](https://github.com/dag-andersen/argocd-diff-preview/pull/16)

## Basic usage in a GitHub Actions Workflow

The most basic example of how to use `argocd-diff-preview` in a GitHub Actions workflow is shown below. In this example, the tool will run on every pull request to the `main` branch, and the diff will be posted as a comment on the pull request.

This example works only if your Git repository is public and you are using public Helm Charts. If you have a private repository or are using private Helm Charts, you need to provide the tool with the necessary credentials. Refer to the [full documentation](https://dag-andersen.github.io/argocd-diff-preview/github-actions-workflow/) to learn how to do this.

```yaml
# .github/workflows/generate-diff.yml
name: Argo CD Diff Preview

on:
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - uses: actions/checkout@v4
        with:
          path: pull-request

      - uses: actions/checkout@v4
        with:
          ref: main
          path: main

      - name: Generate Diff
        run: |
          docker run \
            --network=host \
            -v /var/run/docker.sock:/var/run/docker.sock \
            -v $(pwd)/main:/base-branch \
            -v $(pwd)/pull-request:/target-branch \
            -v $(pwd)/output:/output \
            -e TARGET_BRANCH=${{ github.head_ref }} \
            -e REPO=${{ github.repository }} \
            dagandersen/argocd-diff-preview:v0.0.33

      - name: Post diff as comment
        run: |
          gh pr comment ${{ github.event.number }} --repo ${{ github.repository }} --body-file output/diff.md --edit-last || \
          gh pr comment ${{ github.event.number }} --repo ${{ github.repository }} --body-file output/diff.md
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Helm/Kustomize generated ArgoCD applications
`argocd-diff-preview` will only look for YAML files in the repository with `kind: Application` or `kind: ApplicationSet`. If your applications are generated from a Helm chart or Kustomize template, you will have to add a step in the pipeline that renders the chart/template. Refer to the [full documentation](https://dag-andersen.github.io/argocd-diff-preview/generated-applications/) to learn how to do this.

## Full Documentation

[Link to docs](https://dag-andersen.github.io/argocd-diff-preview/)

## ArgoCon 2024 Talk

<img align="right" src="./docs/assets/ArgoConLogoOrange.svg" width="30%"> `argocd-diff-preview` was presented at ArgoCon 2024 in Utah, US. The talk covered current tools and methods for visualizing code changes in GitOps workflows and introduced this new approach, which uses ephemeral clusters to render accurate diffs directly on your pull requests.

- Talk description: [GitOps Safety: Rendering Accurate ArgoCD Diffs Directly on Pull Requests](
https://colocatedeventsna2024.sched.com/event/1izsL/gitops-safety-rendering-accurate-argocd-diffs-directly-on-pull-requests-dag-bjerre-andersen-visma-regina-voloshin-octopus-deploy)
- Talk recording: [YouTube](https://youtu.be/3aeP__qPSms)

## All Contributors

<a href="https://github.com/dag-andersen/argocd-diff-preview/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=dag-andersen/argocd-diff-preview" />
</a>

## Questions, issues, or suggestions
If you experience issues or have any questions or suggestions, please open an issue in this repository! 🚀
