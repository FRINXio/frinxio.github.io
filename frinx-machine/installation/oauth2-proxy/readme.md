---
expanded: false
order: 1992
---

# Authorization and authentification

Follow official helm chart repository for [oauth2-proxy](https://artifacthub.io/packages/helm/oauth2-proxy/oauth2-proxy).
Don't forget to update the version to a more recent.

```yaml
# Chart.yaml
apiVersion: v2
name: azure-oauth2-proxy
description: Kubernetes deployment of azure-oauth2-proxy
type: application
version: 6.1.0
maintainers:
  - name: FRINX
dependencies:
  - name: oauth2-proxy
    repository: https://oauth2-proxy.github.io/manifests
    version: 7.7.4
    condition: oauth2-proxy.enabled
```

```yaml
# templates/oauth2-proxy-secret.yaml
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: oauth2-proxy
  namespace: frinx
data:
  client-id: <FILL YOUR ID BASE64>
  client-secret: <FILL YOUR SECRET BASE64>
  cookie-secret: <FILL YOUR COOKIE BASE64>
```

+++ Azure AD

Follow [oauth2-proxy official documentation](https://oauth2-proxy.github.io/oauth2-proxy/configuration/providers/azure) to configure Azure AD.

```yaml
# templates/azure-redis-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Chart.Name }}-redis-secret
type: Opaque
data:
  redis-password: {{ .Values.redisPassword.password | b64enc }}
```

```yaml
# values.yaml

x-frinx-image-pull-secret: &frinx-image-pull-secret regcred

oauth2-proxy:
  enabled: true

  fullnameOverride: "oauth2-proxy"

  image:
    repository: "frinxio/oauth2-proxy"
    tag: "6.1.0-alpine"

    imagePullSecrets:
      - name: *frinx-image-pull-secret

  redis:
    enabled: true
    architecture: standalone

  sessionStorage:
    type: redis
    redis:
      existingSecret: "azure-oauth2-proxy-redis-secret"
      passwordKey: "redis-password"

  config:
    existingSecret: oauth2-proxy

    configFile: |-
      # DEFAULT CONFIGURATION
      # https://oauth2-proxy.github.io/oauth2-proxy/configuration/overview

      custom_sign_in_logo = "/tmp/frinx/frinx.png"
      upstreams = "file:///dev/null"

      cookie_secure = true
      cookie_expire = 0
      # cookie_httponly = true

      pass_authorization_header = false
      proxy_websockets = true

      email_domains = [ "*" ]

      # DEPENDS ON DEPLOYMENT SETUP, INGRESS CONFIGURATION
      cookie_domains = [ "fm.127.0.0.1.nip.io" ]
      whitelist_domains = [ "fm.127.0.0.1.nip.io" ]

      provider = "azure"
      azure_tenant = "YOUR_TENANT_ID"
      oidc_issuer_url = "https://login.microsoftonline.com/YOUR_TENANT_ID/v2.0"

      login_url = "https://login.microsoftonline.com"
      redirect_url = "https://fm.127.0.0.1.nip.io/oauth2/callback"

      ssl_insecure_skip_verify = false
      pass_access_token = false
      set_xauthrequest = true
      skip_jwt_bearer_tokens = true
      reverse_proxy = true

  extraArgs:
    azure-graph-group-field: displayName

redisPassword:
  password: "yourPassword"

```

+++


## Install Oauth2-Proxy
```bash
helm dependency update
helm install -n frinx oauth2-proxy . -f values.yaml
```


## Configure RBAC

Rbac functionality can be configured on subchart level.

- https://artifacthub.io/packages/helm/frinx-helm-charts/krakend?modal=values&path=rbac
- https://artifacthub.io/packages/helm/frinx-helm-charts/workflow-manager?modal=values&path=rbac
- https://artifacthub.io/packages/helm/frinx-helm-charts/topology-discovery?modal=values&path=rbac


  env:
    X_AUTH_USER_GROUP: *frinx-rbac-admin-role

```yaml
# values.yaml

https://artifacthub.io/packages/helm/frinx-helm-charts/krakend?modal=values&path=rbac


```