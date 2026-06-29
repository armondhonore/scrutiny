# Nexlayer Fix

## Fixed Dockerfile

```dockerfile
FROM ghcr.io/analogj/scrutiny:master-omnibus
EXPOSE 8080
```

## Fixed nexlayer.yaml

```yaml
application:
  name: scrutiny
  pods:
  - name: app
    image: ghcr.io/analogj/scrutiny:master-omnibus
    path: /
    servicePorts:
    - 8080
```

## Notes

Scrutiny is a hard-drive S.M.A.R.T. monitoring web app. The web component requires an InfluxDB v2 backend.

Root cause of the 503: the deployment used `ghcr.io/analogj/scrutiny:master-web`, which is the web-only image. With no InfluxDB reachable, the web server fails its startup health check / cannot serve, so the route returns 503.

Fix: switch to the maintainer's all-in-one `ghcr.io/analogj/scrutiny:master-omnibus` image, which bundles the web UI, API, SMART collector, and an embedded InfluxDB v2 in a single container. It self-configures on first boot (writes its config and provisions InfluxDB), so no external DB pod or inter-pod DNS is required. The web UI listens on port 8080. The dashboard renders even with no monitored drives (the collector simply finds none in this environment), which is a valid 200.

Do NOT change the image back to `master-web` and do NOT add a separate InfluxDB pod — omnibus is self-contained. Pinned.
