# Guia de Despliegue NEXUS CORE
# =================================================================

## Paso 1: Requisitos Previos

```bash
# Verificar sistema
lsb_release -a
uname -m

# Verificar Docker
docker --version
docker compose version
```

## Paso 2: Seleccionar Perfil

| Perfil | RAM | Servicios |
|---|---|---|
| esencial | 2-4GB | NPM + PG + Redis + Vaultwarden |
| esencial-plus | 4-6GB | + Authentik + PI + OpenLDAP |
| intermedio | 6-8GB | + Odoo |
| intermedio-plus | 8-12GB | + Nextcloud + n8n |
| avanzado | 12GB+ | + Kuma + Wazuh Agent |

## Paso 3: Despliegue

### Opcion A: Interactivo (Recomendado para primer deploy)

```bash
cd /opt/nexus-core
sudo bash core-deploy.sh
```

El script guia paso a paso: dominio, SMTP, SSL, credenciales.

### Opcion B: Automático (Para deploys repetibles)

```bash
# Generar .env primero (una vez)
sudo bash core-deploy.sh --profile intermedio

# Luego desplegar sin interaccion
sudo bash core-deploy.sh --profile intermedio --auto
```

### Opcion C: Flags de gestion de proyectos

```bash
# Crear proyecto con flags
sudo bash core-deploy.sh --new --profile 1 --client "MiCliente" --domain midom.com --auto

# Validar stack post-deploy
sudo bash core-deploy.sh --project core-01-mi-cliente-260626 --validate

# Estado de servicios
sudo bash core-deploy.sh --project core-01-mi-cliente-260626 --logs

# Backup del proyecto
sudo bash core-deploy.sh --project core-01-mi-cliente-260626 --backup

# Destruir proyecto (contenedores + volumenes)
sudo bash core-deploy.sh --project core-01-mi-cliente-260626 --destroy
```

### Opcion D: Solo auditoria

```bash
sudo bash core-deploy.sh --check
```

## Paso 4: Configurar DNS y Proxy

Crear registros DNS tipo A apuntando a la IP del servidor:

| Subdominio | Servicio |
|---|---|
| npm.DOMINIO | NPM Admin |
| auth.DOMINIO | Authentik |
| odoo.DOMINIO | Odoo |
| cloud.DOMINIO | Nextcloud |
| vault.DOMINIO | Vaultwarden |
| n8n.DOMINIO | n8n |
| status.DOMINIO | Uptime Kuma |

Luego acceder a NPM Admin via SSH tunnel para crear Proxy Hosts:

```bash
ssh -L 9090:localhost:81 user@servidor
# Abrir http://localhost:9090
```

## Paso 5: Verificar

```bash
# Validar stack raíz (legacy)
bash scripts/validate_stack.sh

# Validar proyecto específico
bash scripts/validate_stack.sh --project core-01-mi-cliente-260626
```

## Paso 6: Wazuh Agent (solo perfil avanzado)

```bash
sudo bash scripts/wazuh_agent.sh wazuh.tu-soc.com Pyme_Cliente
```

## Comandos Utiles

```bash
# Estado de servicios
docker compose ps

# Logs en tiempo real
docker compose logs -f <servicio>

# Backup manual
bash scripts/make_backup.sh

# Backup por proyecto
bash scripts/make_backup.sh --project core-01-cliente-260626

# Restaurar backup
bash scripts/restore_backup.sh restore/nexus-core-backup-*.tgz

# Generar bundle offline (para VPS sin internet)
sudo bash scripts/offline_resources.sh

# Ciclo de pruebas remoto
bash scripts/test_remote.sh --ip <VPS_IP> --cycle
```

#End Development By Angel Esquivel (CyberSecurity) [NEXUS CORE 2026]
