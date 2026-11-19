<!-- Development By Angel Esquivel (Automation) [NEXUS CORE 2026] -->
# BASELINE — NEXUS CORE (2026-10-21)

## Estado inicial observado

- `core-deploy.sh` ejecutaba `source .env` en despliegue, validacion y modo legacy.
- Generaba `.env` con `chmod 600`, pero imprimia contrasenas en consola.
- `check_docker` instalaba Docker con `curl | sudo bash`.
- `make_backup.sh` producia `.tgz` sin cifrar; `restore_backup.sh` no coincidia
  con los nombres generados y usaba `local` fuera de funcion.
- `docker-compose.yml` usaba `:latest` en NPM, Vaultwarden, n8n y Kuma.
- Kuma publicaba `3001` en `0.0.0.0`; el resto de puertos usaba `BIND_IP`.
- Sin `deploy.sh`, `deploy.conf`, `processes.conf`, `PROMPT.md` ni tests.

## Cambios aplicados

- Parser `scripts/conf.py` (sin shell, sin expansion) y carga segura en
  `core-deploy.sh`; se elimino `eval` de `ask_with_retry`.
- `.env` generado con valores entrecomillados; secretos ya no se imprimen.
- `check_docker` ya no ejecuta instaladores remotos.
- Backup cifrado AES-256-CBC/PBKDF2, manifest SHA-256, retencion y restore
  verificado con `--dry-run`/`--yes`.
- Imagenes principales fijadas por digest; Kuma en `BIND_IP`.
- `cap_drop: ALL` y `pids_limit` en los 12 servicios; `cap_add` minimo.
- Contrato de raiz (`deploy.sh`, `deploy.conf`, `processes.conf`, `bundle.sh`).
- Tests stdlib/Bash sin Docker (`tests/run_all.sh`).

## Deuda pendiente

- Fijar digest de PostgreSQL, Redis, OpenLDAP, Authentik, PrivacyIDEA, Odoo y Nextcloud.
- Probar `docker compose config` y el stack completo en un host con Docker.
- Validar restore real de PostgreSQL/volumenes en laboratorio.
- Migrar `core-deploy.sh` a modulos si supera 2,000 lineas.
<!-- End Development By Angel Esquivel (Automation) [NEXUS CORE 2026] -->
