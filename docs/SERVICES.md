# Glosario de Servicios NEXUS CORE
# =================================================================

## 1. Nginx Proxy Manager (NPM)

Proxy inverso con interfaz web. Maneja SSL/TLS, redirecciones y
distribuye trafico a los servicios internos.

- Puerto: 80/443
- Sin branding (admin tool interna)
- SSL via Let's Encrypt o Cloudflare Origin Certificate

## 2. PostgreSQL 17.10

Base de datos relacional unificada para todos los servicios.

- Una instancia compartida, bases de datos separadas por servicio
- Red `internal_net` aislada (sin acceso a internet)
- Conexiones permitidas solo desde contenedores autorizados

## 3. Redis 7.4

Cache en memoria compartido.

- DB 0: PrivacyIDEA (sesiones, rate-limiting)
- DB 1: Nextcloud (file locking, cache)
- DB 2+: Autenticacion de sesiones (Authentik)

## 4. OpenLDAP

Directorio de usuarios LDAP unificado.

- Fuente de verdad para identidades corporativas
- Authentik lo usa como backend de usuarios
- Odoo y Nextcloud autentican contra LDAP
- PrivacyIDEA lo usa como resolver de usuarios

## 5. Authentik 2026.5.2 (SSO / IdP)

**NUEVO** - Proveedor de Identidad centralizado. Reemplaza la interfaz
de login de PrivacyIDEA como cara visible para el usuario final.

- Single Sign-On (SSO) para todos los servicios
- Branding personalizable desde Admin > Themes
- Policy Engine para reglas de acceso
- Outposts para LDAP, Proxy, RADIUS
- Autenticacion contra OpenLDAP
- MFA delegado a PrivacyIDEA como backend

## 6. PrivacyIDEA 3.12.3 (MFA Backend)

**Backend de autenticacion de dos factores. Invisible para el usuario**
**final (Authentik es la interfaz SSO).**

- Solo administradores acceden al panel de PI
- TOTP-only UI (personalización 42 archivos)
- License bypass incluido
- SMTP sendmail para envio de tokens
- Integrado via API con Authentik

## 7. Odoo 18 CE (ERP/CRM)

Sistema de gestion empresarial.

- PostgreSQL nativo
- Modulos: ventas, inventario, contabilidad, facturacion, CRM, RRHH
- Logo personalizable desde Ajustes > Compania

## 8. Nextcloud 33 (Cloud Corporativo)

Nube privada de archivos, calendarios, contactos y colaboracion.

- App `theming` para branding (logo, colores, nombre)
- Acceso web, movil y desktop
- Colaboracion en tiempo real

## 9. Vaultwarden (Password Manager)

Implementación Rust del servidor Bitwarden. Ligero (~30MB RAM).

- Compatible con todas las apps de Bitwarden
- SIGNUPS_ALLOWED=false (solo invitaciones)
- 2FA via PrivacyIDEA

## 10. n8n (Automatización)

Plataforma de automatización con 400+ conectores.

- Flujos visuales (nodos)
- Webhooks para eventos externos
- PostgreSQL para persistencia

## 11. Uptime Kuma (Monitoreo)

Dashboard de estado para todos los servicios.

- Notificaciones: Email, Telegram, Discord, Slack
- Monitorea HTTP, Ping, DNS, TCP, UDP
- Paginas de estado publicas con branding

## 12. Wazuh Agent (EDR)

Sensor de seguridad en el host. No corre en Docker.

- Audita integridad de archivos
- Detecta malware y anomalias
- Envia alertas a servidor Wazuh central (tu SOC)
- Consumo: ~50MB RAM, <1% CPU

## 13. SMTP Relay

Todos los servicios envian correos via un relay SMTP externo.

- SendGrid (gratis: 100 emails/dia)
- Mailgun (gratis: 1000 emails/mes)
- AWS SES (gratis desde EC2)
- Brevo (gratis: 300 emails/dia)

#End Development By Angel Esquivel (CyberSecurity) [NEXUS CORE 2026]
