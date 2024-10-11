# ExternalDNS - Anexia Webhook Provider

[![License](https://img.shields.io/github/license/probstenhias/external-dns-anexia-webhook?style=for-the-badge)](LICENSE.md)
[![Build](https://img.shields.io/github/actions/workflow/status/probstenhias/external-dns-anexia-webhook/pull_request.yml?style=for-the-badge)](https://github.com/probstenhias/external-dns-anexia-webhook/actions/workflows/pull_request.yml)
[![GoReport](https://goreportcard.com/badge/github.com/probstenhias/external-dns-anexia-webhook?style=for-the-badge)](https://goreportcard.com/report/github.com/probstenhias/external-dns-anexia-webhook)
[![Coverage](https://img.shields.io/coverallsCoverage/github/ProbstenHias/external-dns-anexia-webhook?style=for-the-badge)](https://coveralls.io/github/ProbstenHias/external-dns-anexia-webhook?branch=main)

The Anexia Webhook Provider for [ExternalDNS](https://github.com/kubernetes-sigs/external-dns) allows you to use Anexia's DNS API to manage DNS records for your domains.

The provider is heavily inspired by the [ExternalDNS - IONOS Webhook](https://github.com/ionos-cloud/external-dns-ionos-webhook) and some inspiration taken from the [External DNS - Adguard Home Provider](https://github.com/muhlba91/external-dns-provider-adguard/tree/main).

## Configuration

See [cmd/webhook/init/configuration/configuration.go](cmd/webhook/init/configuration/configuration.go) for all available configuration options for the webhook sidecar, and [internal/anexia/configuration.go](internal/anexia/configuration.go) for all available configuration options for the Anexia provider.

## Kubernetes Deployment

The Anexia Webhook Provider is provided as  an OCI image in [ghcr.io/probstenhias/external-dns-anexia-webhook](https://ghcr.io/probstenhias/external-dns-anexia-webhook).

The following is an example deployment for the Anexia Webhook Provider:

```bash
helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/

# create the anexia configuration
kubectl create secret generic anexia-credentials \
    --from-literal=token='<ANEXIA_API_TOKEN>'

# create the helm values file
cat <<EOF > external-dns-anexia-values.yaml

# -- ExternalDNS Log level.
logLevel: debug # reduce in production

# -- if true, _ExternalDNS_ will run in a namespaced scope (Role and Rolebinding will be namespaced too).
namespaced: false

# -- _Kubernetes_ resources to monitor for DNS entries.
sources:
  - ingress
  - service
  - crd

extraArgs:
  ## must override the default value with port 8888 with port 8080 because this is hard-coded in the helm chart
  - --webhook-provider-url=http://localhost:8080

provider:
  name: webhook
  webhook:
    image: ghcr.io/probstenhias/external-dns-anexia-webhook
    tag: v0.1.6
    env:
      - name: LOG_LEVEL
        value: debug # reduce in production
      - name: ANEXIA_API_URL
        value: <ANEXIA_API_URL>
      - name: ANEXIA_API_TOKEN
        valueFrom:
          secretKeyRef:
            name: anexia-credentials
            key: token
      - name: SERVER_HOST
        value: "0.0.0.0"
      - name: DRY_RUN
        value: "false"
EOF

# install external-dns with helm
helm upgrade -i external-dns-anexia external-dns/external-dns -f external-dns-anexia-values.yaml
```
