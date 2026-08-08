# Ciclo de Pruebas NEXUS CORE - Pipeline de Validación
# =================================================================
# Pipeline completo de validación para proyectos cliente.
# Detalle técnico: docs/TECHNICAL.md
# Comandos operativos: docs/RUNBOOK.md
# Automatización: scripts/test_remote.sh
# =================================================================

## Pipeline Principal (Orden de Ejecución)

| Paso | Fase | Que Validar | Script/Accion | Criterio Exito |
|------|------|-------------|---------------|----------------|
| PT | Pre-Test | Detectar recursos VPS | `scripts/pre_test.sh` | Muestra RAM, CPU, disco, perfil maximo recomendado |
| P0 | Help | Flags --help, --version | `core-deploy.sh --help` | Muestra menu completo de flags + 5 perfiles con descripción |
| P1 | Crear Proyecto | Nuevo proyecto perfil 1 con cliente | `--new --profile 1 --client TestA` | Crea `projects/core-01-testa-<DDMMYY>/` con .env, branding/, context.txt |
| P2 | Perfil Sin Cliente | Perfil 3 sin nombre cliente | `--new --profile 3` | Crea `projects/core-03-<DDMMYY>/` sin CLIENT_NAME |
| P3 | Validación Input | 3 intentos fallidos en perfil | Menu > Nuevo Proyecto > perfil invalido x3 | Retorno seguro al menu principal sin crash |
| P4 | Listar Proyectos | --list con proyecto existente | `--list` | Muestra proyecto con estado [detenido] |
| P5 | Deploy Esencial | Deploy proyecto perfil 1 | `--project core-01-* --deploy` | Contenedores up, healthchecks OK |
| P6 | Validar Stack | validate_stack.sh post-deploy | `--project core-01-* --validate` | 12+ health checks, todos PASS |
| P7 | Estado Servicios | docker compose ps | `--project core-01-* --logs` o menu accion 4 | Servicios running sin errores en logs |
| P8 | Destruir Proyecto | down -v post-pruebas | `--project core-01-* --destroy` | Contenedores eliminados, volumenes limpios |
| P9 | Perfil Intermedio | Crear + deploy perfil 3 con cliente | `--new --profile 3 --client TestB --auto` | 7+ servicios, validación completa |
| P10 | Auditoria Check | Trivy + recursos | `--check` | Reporte generado, 0 errores criticos, recurso detectado |
| P11 | Offline Bundle | offline_resources.sh | `bash scripts/offline_resources.sh` | .deb + imagenes Docker + guia generados en restore/offline_bundle/ |

## Fases de Regresion

| Paso | Fase | Cuando se ejecuta | Que valida |
|------|------|-------------------|------------|
| R1 | Destruir y recrear | Post-P8 | Proyecto existente se destruye, se crea uno nuevo con mismo perfil |
| R2 | Multi-proyecto | Post-P9 | Dos proyectos coexisten sin conflictos de puertos/redes |
| R3 | Backup/Restore | Post-P6 | make_backup.sh genera .tgz, restore_backup.sh restaura | 

## Reglas de Ejecución

1. **P0 antes de P1**: Verificar que help funciona antes de cualquier operación
2. **P4 antes de P5**: Listar confirma que el proyecto existe antes de desplegar
3. **P6 inmediatamente despues de P5**: Validar salud post-deploy sin demora
4. **P8 al final del ciclo**: Destruir solo cuando todas las validaciones P0-P7 pasaron
5. **PT antes de P0**: Siempre ejecutar pre_test.sh para detectar recursos antes de testing
6. **P11 opcional**: Si el VPS destino tiene internet, offline bundle no es necesario
7. **R3 opcional**: Backup/Restore se prueba solo si hay almacenamiento disponible

## Modo Testing (Sin SSL)

Para VPS con IP privada o sin dominio asignado, usar `SSL_MODE=3` (Testing):
- Acceso directo por IP:puerto sin certificados
- Todos los servicios expuestos en `0.0.0.0` para debugging
- Ideal para testing en laboratorio o VPS internos

```bash
# Crear proyecto en modo testing
bash core-deploy.sh --new --profile 1 --client TestA --testing

# Esto genera .env con TESTING_MODE=true y BIND_IP=0.0.0.0
# Los servicios son accesibles via http://IP:puerto
```

**Accesos en modo Testing:**
| Servicio | Puerto | URL |
|----------|--------|-----|
| NPM Admin | 81 | http://IP:81 |
| Authentik | 9000 | http://IP:9000 |
| Odoo | 8069 | http://IP:8069 |
| Uptime Kuma | 3001 | http://IP:3001 |
| PostgreSQL | 5432 | IP:5432 (solo debug) |
| Redis | 6379 | IP:6379 (solo debug) |

## Variables de Entorno (Rápido)

| Variable | Fuente | Descripción |
|----------|--------|-------------|
| VPS_IP | Statico / variable | IP del servidor remoto de pruebas |
| VPS_USER | Statico / variable | Usuario SSH para conexion remota |
| VPS_PATH | `/opt/nexus-core` | Ruta del proyecto en VPS remoto |
| PROFILE_TEST | 1 o 3 | Perfil usado en la fase activa |
| CLIENT_TEST | TestA o TestB | Cliente usado en la fase activa |
| PROJECT_FOLDER | Generado por core-deploy.sh | `core-<NN>-<cliente>-<DDMMYY>` |

## Troubleshooting Rápido

| Sintoma | Causa Probable | Referencia |
|---------|---------------|------------|
| core-deploy.sh no arranca | Permisos de ejecución | `chmod +x core-deploy.sh` |
| docker compose build falla | Falta .env o variables | Verificar `.env.template` |
| Contenedor no sube | Puerto ocupado | `lsof -i :<puerto>` |
| Healthcheck FAIL | Servicio no ready | Esperar 30s + `docker logs <container>` |
| validate_stack.sh falla todo | Ningun contenedor running | Revisar `docker compose ps --all` |
| Trivy reporta errores | USO de root en Docker | Ignorar si es test; revisar en prod |
| offline_bundle vacio | Sin imagenes buildedas | Ejecutar `docker compose build` primero |

## Infraestructura de Pruebas

| Elemento | Ubicacion | Proposito |
|----------|-----------|-----------|
| Restore backups | `restore/` | Almacenamiento de backups de prueba |
| Offline bundles | `restore/offline_bundle/` | Paquetes offline generados |
| Logs de pruebas | `logs/` (creado por test_remote.sh) | Resultados de cada fase |

#End Development By Angel Esquivel (CyberSecurity) [NEXUS CORE 2026]
