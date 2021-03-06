# Running Dracon Demo Pipeline Against a Private Repository via Git+SSH

## Prerequisites

- You have followed the [Getting Started with Minikube Guide](/docs/getting-started/minikube.md)

---

## Tutorial

1. Clone the Dracon repository.

   ```bash
   $ git clone https://github.com/thought-machine/dracon.git "${PWD}/dracon"
   ```

2. Copy an example demo pipeline from the `./examples/pipelines` directory into your working directory. We have chosen `mixed-lang-project`.

   ```bash
   $ cp -r "${PWD}/dracon/examples/pipelines/mixed-lang-project" "${PWD}"
   ```

3. Add the following resources to the `pipeline-run.yaml` and do the following:

   1. Replace the `FILL_ME_IN`s with the guidance.
   2. Name the resources consistently, we've opted for `gitssh-<repository_domain>-<repository_path_kebab_separated>`, examples:
      - `github.com/thought-machine/dracon` -> `gitssh-github-thought-machine-dracon`
      - `github.com/tektoncd/pipeline` -> `gitssh-github-tektoncd-pipeline`
   3. Set the `tekton.dev/v1alpha1, PipelineRun` resource's `spec.serviceAccountName` to the `v1, ServiceAccount` you just added.
   4. Remove the previous `tekton.dev/v1alpha1, PipelineResource`.

      ```yaml
      ---
      apiVersion: v1
      kind: Secret
      metadata:
        name: gitssh-github-tektoncd-pipeline
        annotations:
          tekton.dev/git-0: github.com
        labels: {}
      type: kubernetes.io/ssh-auth
      data:
        # Generated by:
        # cat id_rsa | base64 -w 0
        # This deploy key has read-only permissions on github.com/knative/build
        ssh-privatekey: FILL_ME_IN
        # Generated by:
        # ssh-keyscan github.com | base64 -w 0
        known_hosts: FILL_ME_IN
      ---
      apiVersion: v1
      kind: ServiceAccount
      metadata:
        name: gitssh-github-tektoncd-pipeline
        labels: {}
      secrets:
        - name: gitssh-github-tektoncd-pipeline
      ---
      # git+ssh config: pipeline resource
      apiVersion: tekton.dev/v1alpha1
      kind: PipelineResource
      metadata:
        name: "{{.RunID}}-gitssh-github-tektoncd-pipeline"
        labels: {}
      spec:
        type: git
        params:
          - name: revision
            value: master
          - name: url
            value: git@github.com:tektoncd/pipeline.git
      ```

4. You can now run `dracon setup` and `dracon run` with your pipeline

   ```bash
   $ dracon setup --pipeline mixed-lang-project
   $ dracon run --pipeline mixed-lang-project
   ```
