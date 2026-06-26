# appchart

a simple chart that ease deployment of classic apps on [my k8s cluster, lampone](https://github.com/k0rventen/lampone).

The majority of apps deployed on my cluster are a simple container, exposing a service, with a small conf, and maybe some persistent storage or secret. Instead of deploying full-fledged YAMLs for every single one of them, and repeating myself, this chart requires very few lines to deploy this kind of app.  

**A lot of flags are specific to my deployment, and use specifics of my cluster (the ingressclass/storageclass names for example).**


## Very basic example

Declare the HR somewhere

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: appchart
  namespace: flux-system
spec:
  interval: 24h
  url: https://k0rventen.github.io/appchart/
```

Then your HelmRelease:

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: example
spec:
  chart:
    spec:
      chart: appchart
      version: "x.x.x"
      sourceRef:
        kind: HelmRepository
        name: appchart
        namespace: flux-system
      interval: 10m
  targetNamespace: services
  values:
    apps:
      # name of the deployment
    - name: simple-echo
      # image to use
      image: traefik/whoami@sha256:200689790a0a0ea48ca45992e0450bc26ccab5307375b41c84dfc4f2475937ab
      # port exposed by the container
      service: 80
      # domain to access the service
      ingress:
        domain: echo.domain.com
      # env vars
      env:
        - name: key
          value: val
```

## more complex usage

See `example-values.yaml` for a complete example of all possible values


## testing

```
helm template -f example-values.yaml . | k apply -f - --dry-run=server
```


## changelog

1.8.0: default values are now in `values.yaml`, reworked the middleware mgmt, removed unused `secrets`.

1.7.0: renamed `ingress.geoblock` to `ingress.worldwide_access` to avoid false value being ommited and being more explicit

1.6.0: added `ingress.geoblock` for traefik middleware flag, updated anubis image

1.5.0: rename `extras -> containerSpec`, add `podSpec`, and `secrets` for generating simple secrets

1.4.1: chore bump for triggering the CI for the first time

1.4.0: add `deploymentSpec` key for adding extra config at the deployment level

1.3.0: add annotation so flux avoid reconciliation on ingresses when modified by klipper

1.2.0: added `extras` key that allows settings arbitrary container-level specs
