---
expanded: false
order: 9999
label: Docker Registry Secret
---

# Create Docker Registry Secret

Create a kubernetes Docker registry secret for pulling images from private registry:

```bash
# PLACEHOLDERS must be replaced with user credentials
kubectl create secret -n frinx docker-registry regcred \
    --docker-server="https://index.docker.io/v1/" \
    --docker-username="<PLACEHOLDER>" \
    --docker-password="<PLACEHOLDER>"
```

For more info about accessing private images, visit [Download Frinx Uniconfig](https://docs.frinx.io/frinx-uniconfig/getting-started/#download-frinx-uniconfig)

