# Arquitectura Técnica y Redes NEXUS CORE
# =================================================================

## Stack

- Proxy: Nginx Proxy Manager (80/443)
- Identity: Authentik (SSO/MFA) + OpenLDAP
- 2FA: PrivacyIDEA (TOTP-only UI)
- ERP: Odoo
- Cloud: Nextcloud
- Password Manager: Vaultwarden
- Automation: n8n
- Monitoring: Uptime Kuma
- Host Security: Wazuh Agent

## Capas De Red

| Capa | Red Docker | Servicios | Acceso Internet |
|---|---|---|---|
| Perimetral | docker_net | NPM, Authentik | Si (puertos 80/443) |
| Aplicaciones | docker_net | Odoo, Nextcloud, Vaultwarden, n8n, Kuma | Solo entrante |
| Datos | internal_net | PostgreSQL, Redis, OpenLDAP, PrivacyIDEA | NO |
| Host | N/A | Wazuh Agent, Trivy | Si (reportes SOC) |

## Orquestador

`core-deploy.sh` gestiona proyectos en `projects/core-<NN>-<cliente>-<DDMMYY>/`
con .env propio, branding, perfil 1-5, y SSL unificado.

## Perfiles

| Perfil | RAM | Servicios |
|---|---|---|
| esencial | 2-4GB | NPM + PG + Redis + Vaultwarden |
| esencial-plus | 4-6GB | + Authentik + PI + OpenLDAP |
| intermedio | 6-8GB | + Odoo |
| intermedio-plus | 8-12GB | + Nextcloud + n8n |
| avanzado | 12GB+ | + Kuma + Wazuh Agent |

- Licensing: D3 — Source Available (BSL + Commercial)

#End Development By Angel Esquivel (CyberSecurity) [NEXUS CORE 2026]
