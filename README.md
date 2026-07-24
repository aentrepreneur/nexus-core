# NEXUS CORE - Infraestructura Blindada para PYMEs LATAM
# =================================================================

Stack modular de servicios open source bajo proxy inverso unificado con
SSO, MFA, ERP, Cloud, Password Manager y Automatización.

## Arquitectura General

Proxy inverso (NPM) -> Authentik (SSO/MFA) -> Servicios (Odoo, Nextcloud,
Vaultwarden, n8n, Kuma) -> Base de datos unificada (PostgreSQL + Redis)

## Requisitos

- Ubuntu 24.04
- Docker + Docker Compose
- 2-12 GB RAM (segun perfil)
- Dominio configurado (Cloudflare o DNS directo)

## Despliegue Rapido

```bash
git clone <repo>
cd nexus-core
./core-deploy.sh
```

Seleccionar perfil 1-5 y seguir el asistente interactivo.

## Documentacion

| Documento | Proposito |
|---|---|
| docs/AGENTS.md | Fuente de verdad arquitectonica |
| docs/TECHNICAL.md | Arquitectura tecnica y redes |
| docs/SERVICES.md | Glosario de servicios |
| docs/DEPLOYMENT.md | Guia de despliegue |
| docs/CHANGELOG.md | Historial de versiones |

## Licencia

D3 — Source Available (BSL + Commercial). Ver LICENSE.

#End Development By Angel Esquivel (CyberSecurity) [NEXUS CORE 2026]
