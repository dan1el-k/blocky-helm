# Blocky DNS Proxy - Helm Chart

Deploys [Blocky DNS Proxy](https://github.com/0xERR0R/blocky) together with [K8s-Gateway](https://github.com/ori-edge/k8s_gateway) for automated Split-DNS Setup for your local (public) domain.
A [Frontend](https://github.com/Mozart409/blocky-frontend) can be enabled optionally


Releases are provided as a Helm chart artifacts for the sake of easier configuration possibilities, but could be also consumed easily via kustomize and GitOps.


## Example how to use it

### Pull in the Chart via Kustomize into your GitOps AppCatalog

```yaml
#kustomization.yaml

helmCharts:
  - name: blocky-helm
    includeCRDs: true
    releaseName: blocky-helm
    namespace: blocky-helm
    version: 1.0.1
    repo: https://dan1el-k.github.io/blocky-helm
    valuesFile: values.yaml

```

### Example Values for local Domain and SplitDNS setup

```yaml
values.yaml
#####################################################################################
### blocky SubChart
#####################################################################################

service:
  enabled: true
  type: LoadBalancer
  loadBalancerIP: "<provided-dns-ip-via-lb>"

# See https://0xerr0r.github.io/blocky/v0.23/configuration/ for all available options
config:

  upstreams:
    groups:
      default:
        #- 1.1.1.1
        #- 8.8.8.8
        # DNS over HTTPS upstreams
        - https://1.1.1.1/dns-query
        - https://1.0.0.1/dns-query
        - https://dns.google/dns-query
        # DNS over TLS upstreams
        #- tcp-tls:dns.quad9.net
    # blocky will pick the 2 fastest upstreams but you can also use the `strict` strategy
    strategy: parallel_best
    timeout: 2s

  blocking:
    # I prefer the HaGeZi Light blocklist for set and forget setup, you can use any other blacklist nor whitelist you want
    # Blocky supports hosts, domains and regex syntax
    blackLists:
      ads:
        - https://raw.githubusercontent.com/hagezi/dns-blocklists/main/domains/light.txt
    clientGroupsBlock:
      default:
        - ads
    blockType: zeroIp
    blockTTL: 1m
    loading:
      refreshPeriod: 6h
      downloads:
        timeout: 60s
        attempts: 5
        cooldown: 10s
      concurrency: 16
      # start answering queries immediately after start
      strategy: fast
      maxErrorsPerSource: 5
  caching:
    # enable prefetching improves performance for often used queries
    prefetching: true
    # if a domain is queried more than prefetchThreshold times, it will be prefetched for prefetchExpires time
    prefetchExpires: 24h
    prefetchThreshold: 2

  # use encrypted DNS for resolving upstreams
  # same syntax as normal upstreams
  bootstrapDns:
    - 1.1.1.1


ui:
  enabled: true
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: traefik
      kubernetes.io/tls-acme: "true"


#####################################################################################
### k8s-gateway SubChart
#####################################################################################
k8sGateway:
  enabled: true

  watchedResources:
    - "Ingress"
    - "Service"
  
  domain: <local-domain>

  # Optional configuration option for DNS01 challenge that will redirect all acme
  # challenge requests to external cloud domain (e.g. managed by cert-manager)
  # See: https://cert-manager.io/docs/configuration/acme/dns01/
  dnsChallenge:
    enabled: true
    domain: dns01.clouddns.com

```


## Additional links

## Recommended blocklist

- https://github.com/hagezi/dns-blocklists?tab=readme-ov-file#pro



## Credits

Based on the Originals Helm Chart from 
https://github.com/philipfreude/blocky-helm and the values and split-DNS setup from [TrueCharts community](https://github.com/truecharts/public/blob/master/charts/premium/blocky/values.yaml)