# Blocky DNS Proxy - Helm Chart

Deploys [Blocky DNS Proxy](https://github.com/0xERR0R/blocky) together with [K8s-Gateway](https://github.com/ori-edge/k8s_gateway) for automated Split-DNS Setup for your local (public) domain.
A [Frontend](https://github.com/Mozart409/blocky-frontend) can be enabled optionally


Releases are provided as a Helm chart artifacts for the sake of easier configuration possibilities, but could be also consumed easily via kustomize and GitOps.


## Example how to use it

### Pull in the Chart via Kustomize into your GitOps AppCatalog

```yaml
#kustomization.yaml

helmGlobals:
 chartHome: charts

helmCharts:
  - name: blocky
    includeCRDs: true
    releaseName: blocky
    namespace: blocky
    version: 0.6.1
    valuesFile: values.yaml

```



## Additional links

## Recommended blocklist

- https://github.com/hagezi/dns-blocklists?tab=readme-ov-file#pro



## Credits

Based on the Originals Helm Chart from 
https://github.com/philipfreude/blocky-helm and the values and split-DNS setup from [TrueCharts community](https://github.com/truecharts/public/blob/master/charts/premium/blocky/values.yaml)