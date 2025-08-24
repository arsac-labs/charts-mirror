# OCI Helm Charts Mirror

This is our stop-gap mirror of OCI Helm Charts that can be used until maintainers of upstream charts publish them. See the issue [here](https://github.com/home-operations/charts-mirror/issues/8) for tracking the progress of upstream support for OCI charts added here.

> [!CAUTION]
> **Subscribe to the upstream issues or PRs tracking OCI support** because if you wish to use these charts understand it is **your responsiblity to make sure to change to the official OCI chart as soon as possible** as they will be deprecated here. I bear **no resposibility** for you **not paying close attention to this repository and the changes herein**. Once there is support upstream the OCI charts will remain published to this repo for 6 months, after which they will be pruned.

## Usage

### CLI

```sh
helm install ${RELEASE_NAME} --namespace ${NAMESPACE} oci://ghcr.io/home-operations/charts-mirror/${CHART_NAME} --version ${CHART_VERSION}
```

### Flux

> [!WARNING]
> Even though these charts are signed via cosign it will not prevent against malicious code being pushed from upstream ending up in a release here. For example if cert-managers Helm chart is compromised, there's nothing stopping that release from **NOT** being mirrored here.

```yaml
---
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: OCIRepository
metadata:
  name: ${CHART_NAME}
  namespace: ${NAMESPACE}
spec:
  interval: 1h
  layerSelector:
    mediaType: application/vnd.cncf.helm.chart.content.v1.tar+gzip
    operation: copy
  ref:
    tag: ${CHART_VERSION}
  url: oci://ghcr.io/home-operations/charts-mirror/${CHART_NAME}
  verify:
    provider: cosign
    matchOIDCIdentity:
      - issuer: ^https://token.actions.githubusercontent.com$
        subject: ^https://github.com/home-operations/charts-mirror.*$
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: ${RELEASE_NAME}
  namespace: ${NAMESPACE}
spec:
  interval: 1h
  chartRef:
    kind: OCIRepository
    name: ${CHART_NAME}
    namespace: ${NAMESPACE}
  values:
...
```

## Contributing

To add a new chart to this repository:

1. **Check for an existing OCI Helm Chart**

   Confirm that the application you want to add does **not** already provide an official OCI Helm Chart.

2. **Create a chart directory**

   Make a new directory under `apps/` named after the chart.

3. **Add chart metadata**

   Inside the new directory, create a `metadata.yaml` file with the following structure.
   Update the fields to match the upstream chart details:

   ```yaml
   ---
   chartRegistry: <Registry of the upstream chart>
   chartName: <Name of the upstream chart>
   chartVersion: <Version of the upstream chart>
   artifactName: <Name of the published chart>
   ```

4. **Request upstream OCI support**

   If the upstream project does not yet publish OCI Helm Charts, open an issue in their application or chart repository requesting OCI Helm Chart support.

5. **Submit a pull request**

   Open a PR in this repository:
   - Include the link to the upstream issue (from step 4) in the PR description.
   - Ensure your PR only adds the new chart directory and metadata.

## Maintaining a Fork

Forking this repository is fairly straightforward, but there are a couple of important notes:

1. You’ll need to set up a GitHub Bot for Renovate, you can find instructions for that outlined [here](https://github.com/renovatebot/github-action).

2. If your GitHub username or the repository name includes uppercase letters, you’ll need to update the workflows. This is because pushing to GHCR requires both the username and repository name to be entirely lowercase.
