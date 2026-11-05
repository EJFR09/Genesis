---
status: open
priority: normal
dateCreated: 2026-10-28T07:51:17.807-03:00
dateModified: 2026-10-28T07:51:42.890-03:00
tags:
  - task
---
podman ps -a \
  --format 'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}'
ls -ld /app/payment/claro-payment-api/logs
ls -lah /app/payment/claro-payment-api/logs
ls -lah /app/payment/claro-payment-api/logs/logs