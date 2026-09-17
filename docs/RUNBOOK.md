# Runbook Operativo NEXUS CORE - Comandos de Prueba Remota
# =================================================================
# Comandos exactos para cada fase del ciclo de pruebas.
# Uso: source este archivo y ejecutar fases individualmente.
# Automatización: bash scripts/test_remote.sh
# =================================================================

## Variables de Entorno (Ajustar por VPS destino)

```bash
VPS_IP="172.16.5.190"
VPS_USER="angelo"
VPS_PATH="/opt/nexus-core"
LOCAL_PATH="/opt/nexus-core"
```

## Pre-Vuelo: Rsync del Proyecto al VPS

```bash
# Sincronizar código fuente (excluye .env, data, venv)
rsync -avz --delete \
  --exclude='.env' --exclude='data/' --exclude='venv/' \
  --exclude='.git/' --exclude='__pycache__/' \
  --exclude='projects/*/.env' \
  -e ssh "$LOCAL_PATH/" "$VPS_USER@$VPS_IP:$VPS_PATH/"
```

## PT: Pre-Test (Detectar Recursos)

```bash
ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && bash scripts/pre_test.sh"
# Esperado: RAM, CPU, disco, perfil maximo recomendado
# Ejemplo: "PERFIL MAXIMO RECOMENDADO: 2"
```

## Modo Testing (Sin SSL / IP Privada)

Para testing en VPS sin dominio o con IP privada:

```bash
# Crear proyecto en modo testing (sin certificados)
ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && bash core-deploy.sh --new --profile 1 --client TestA --testing --auto"

# Esto genera .env con TESTING_MODE=true y BIND_IP=0.0.0.0
# Los servicios son accesibles via http://IP:puerto
```

**Accesos directos en modo Testing:**
| Servicio | Puerto | Comando de prueba |
|----------|--------|-------------------|
| NPM Admin | 81 | `curl -s http://$VPS_IP:81` |
| Authentik | 9000 | `curl -s http://$VPS_IP:9000` |
| Odoo | 8069 | `curl -s http://$VPS_IP:8069` |
| Uptime Kuma | 3001 | `curl -s http://$VPS_IP:3001` |

## Fases del Pipeline

### P0: Verificar Help

```bash
ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && bash core-deploy.sh --help"
# Esperado: menu completo con 5 perfiles, flags, ejemplos
```

### P1: Crear Proyecto Perfil 1 con Cliente

```bash
ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && bash core-deploy.sh --new --profile 1 --client TestA"
# Esperado: crea projects/core-01-testa-<DDMMYY>/
```

### P2: Crear Proyecto Perfil 3 Sin Cliente

```bash
ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && bash core-deploy.sh --new --profile 3"
# Esperado: crea projects/core-03-<DDMMYY>/
```

### P3: Validación de Input (3 Intentos Fallidos)

```bash
# No automatizable por SSH directamente.
# Ejecutar interactivamente en VPS:
ssh -t "$VPS_USER@$VPS_IP" "cd $VPS_PATH && bash core-deploy.sh"
# Menu > 1) Nuevo Proyecto > Perfil: 6 (invalido) x3
# Esperado: vuelve al menu principal
```

### P4: Listar Proyectos

```bash
ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && bash core-deploy.sh --list"
# Esperado: muestra core-01-testa-* [detenido] y core-03-* [detenido]
```

### P5: Deploy Proyecto Esencial

```bash
ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && bash core-deploy.sh --project core-01-testa-* --deploy"
# Esperado: contenedores up, healthchecks pasando
# Verificar:
ssh "$VPS_USER@$VPS_IP" "docker ps --format 'table {{.Names}}\t{{.Status}}'"
```

### P6: Validar Stack Post-Deploy

```bash
ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && bash core-deploy.sh --project core-01-testa-* --validate"
# Esperado: 12+ health checks, todos PASS
```

### P7: Estado y Logs de Servicios

```bash
# Estado via --logs (muestra docker compose ps)
ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && bash core-deploy.sh --project core-01-testa-* --logs"

# Logs en tiempo real (5 segundos)
ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && timeout 5 docker compose --env-file projects/core-01-testa-*/.env logs --tail=20"
```

### P8: Destruir Proyecto

```bash
ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && bash core-deploy.sh --project core-01-testa-* --destroy"
# Esperado: contenedores y volumenes eliminados
```

### P9: Perfil Intermedio con Auto Deploy

```bash
ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && bash core-deploy.sh --new --profile 3 --client TestB --auto"
# Esperado: crea y despliega proyecto con 7+ servicios
```

### P10: Auditoria Preventiva

```bash
ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && bash core-deploy.sh --check"
# Esperado: reporte Trivy + recursos del servidor
```

### P11: Generar Offline Bundle

```bash
ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && sudo bash scripts/offline_resources.sh"
# Esperado: restore/offline_bundle/ con .deb + imagenes + guia
```

## Post-Vuelo: Recolectar Resultados

```bash
# Traer resultados de vuelta a maquina local
scp -r "$VPS_USER@$VPS_IP:$VPS_PATH/restore/offline_bundle/" "$LOCAL_PATH/restore/"

# Logs de prueba (si test_remote.sh los genero)
scp -r "$VPS_USER@$VPS_IP:$VPS_PATH/logs/" "$LOCAL_PATH/"
```

## Fases de Regresion

### R1: Destruir y Recrear Mismo Proyecto

```bash
PROJECT=$(ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && ls projects/ | grep core-01- | head -1")
ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && bash core-deploy.sh --project $PROJECT --destroy"
ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && bash core-deploy.sh --new --profile 1"
```

### R2: Multi-Proyecto Sin Conflictos

```bash
# Verificar que proyectos no comparten puertos en conflicto
ssh "$VPS_USER@$VPS_IP" "ss -tlnp | grep -E ':(80|443|5432|6379|9000|8000|8069|5678|3001)'"
# Esperado: solo un proceso por puerto
```

### R3: Backup y Restore

```bash
ssh "$VPS_USER@$VPS_IP" "cd $VPS_PATH && bash scripts/make_backup.sh"
# Verificar que el backup se creo
ssh "$VPS_USER@$VPS_IP" "ls -lh $VPS_PATH/restore/*.tgz"
```

## Diagnostico

| Comando | Que Revisa |
|---------|------------|
| `docker ps -a` | Todos los contenedores y su estado |
| `docker logs <container> --tail=30` | Ultimas 30 líneas de log |
| `docker compose --env-file projects/*/.env ps` | Estado del stack del proyecto |
| `free -m` | Memoria disponible |
| `df -h /` | Disco disponible |
| `ss -tlnp` | Puertos en escucha |

#End Development By Angel Esquivel (CyberSecurity) [NEXUS CORE 2026]
