<!-- ================================================================= -->
<!-- Development By Angel Esquivel (Automation) [NEXUS-CORE 2026] -->
<!-- ================================================================= -->
<!-- NEXUS-CORE - Nucleo compartido del ecosistema Nexus -->

# Changelog
# =================================================================

## 2026-10-21

### Security
- Pin {LDAP_VERSION:-1.5.0} en docker-compose.yml (antes tag suelto 1.5.0)
- Auditoria /opt/* global: archivos .env con modo 644 corregidos a 600 (stix-template, socar-template, misp-devel, linkedin-engine, gt-devtools, music-online, opencti-docker, ai-monetize, socar-reports, imar-template, repos-github)
- Auditoria /opt/ai-monetize: postgres:16-alpine y redis:7-alpine
  ahora usan version vars {POSTGRES_VERSION:-16.9-alpine} y
  {REDIS_VERSION:-7.4.9-alpine}

### Changed
- patron de versiones coherente entre nexus-core y ai-monetize

## 2026-07-01
- Licensing dimension assigned: D3 — Source Available (BSL + Commercial)
- Implemented LICENSE, EULA, CONTRACT, AUTHORS with template injection
- Added license-fingerprint.sh: SHA256 hardware fingerprint + JSON output
- Added license-revoke.sh: kill-switch con validacion local y remota
- Added .fingerprint.json, .license.json, .revoked.lock to .gitignore
- Initial project scaffold for nexus-core

<!-- End Development By Angel Esquivel (Automation) [NEXUS-CORE 2026] -->
