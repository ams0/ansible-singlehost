# Using Traefik as reverse proxy

Please update this snippet in `traefik/dynamic.yaml`:

```
  routers:
    # HTTPS routers for all domains
    backend:
      rule: "Host(`dash.vps.kubespaces.cloud`) || `n8n.vps.kubespaces.cloud`) || Host(`s3.vps.kubespaces.cloud`) || Host(`code.vps.kubespaces.cloud`) || Host(`minecraft.vps.kubespaces.cloud`) || Host(`echo.vps.kubespaces.cloud`) || Host(`webhook.vps.kubespaces.cloud`) || Host(`helix.vps.kubespaces.cloud`) || Host(`rancher.vps.kubespaces.cloud`)"
```

adding the host you want to serve using GatewayAPI.
