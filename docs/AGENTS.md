# AGENTS.md - Fuente De Verdad Arquitectonica NEXUS CORE
# =================================================================
# Este archivo es la autoridad en reglas, flujo y seguridad del
# proyecto. Todo documento derivado debe referenciar este archivo.
# =================================================================

## Identidad Del Proyecto

Stack modular de servicios open source para PYMEs LATAM bajo proxy
inverso unificado con SSO, MFA, ERP, Cloud, Password Manager y
Automatización.

Repositorio: https://github.com/aentrepreneur
Entorno: Ubuntu 24.04 + Docker + VPS Cliente

---

## Reglas De Seguridad

1. **PROHIBIDO** `shell=True`, `os.system()`, `os.popen()`, `subprocess.Popen(shell=True)`
   - USAR SIEMPRE: `asyncio.create_subprocess_exec(*args)`
   - Razon: Previene inyeccion de código

2. **PROHIBIDO** hardcodear secretos, API keys o contrasenas en el código.
   - USAR SIEMPRE: `.env` con `chmod 600`
   - Razon: Seguridad

3. **PROHIBIDO** logging de API keys, tokens, contrasenas o contenido de mensajes.

4. **Red internal_net** con `internal:true`. Solo contenedores autorizados
   acceden a PostgreSQL, Redis, OpenLDAP, PrivacyIDEA.

5. **Wazuh Agent** se instala en host, NO en Docker. Reporta a servidor
   Wazuh central (infraestructura del usuario).

---

## Reglas De Dependencias Y Despliegue

6. **PROHIBIDO** agregar tecnologias, dependencias o cambios arquitectonicos
   sin consultar al usuario.

7. **Cada proyecto en su propio VPS**: projects/core-<NN>-<cliente>-<DDMMYY>/
   con .env propio. Un proyecto por servidor.

8. **Perfiles**: 1=esencial, 2=esencial-plus, 3=intermedio, 4=intermedio-plus,
   5=avanzado. Cada perfil suma servicios al anterior.

9. **docker-compose.yml** es el template base compartido. Cada proyecto
   usa `--env-file` para inyectar su .env propio.

10. **Dockerfile custom** solo para PrivacyIDEA (overlay 42 archivos) y
    Odoo (config injection). NPM, Nextcloud, Vaultwarden, n8n usan
    imagenes oficiales.

---

## Reglas De Consistencia

11. **PROHIBIDO** usar `datetime.utcnow()` sin zona horaria.
    - USAR SIEMPRE: `now_local()` o `datetime.now(timezone.utc)`

12. **PROHIBIDO** usar caracteres no latinos, emojis, emoticones o
    simbolos extranos. Solo ASCII puro.

13. **TZ** por defecto: America/Mexico_City (configurable en .env)

---

## Reglas De Calidad

14. **PROHIBIDO** placeholders, TODOs o funciones vacías en código entregado.
    Cada función debe ser ejecutable tal como esta.

15. **Docstrings obligatorios** en toda función pública Python con formato
    Args/Returns/Raises.

16. **Comentarios estilo PascalCase Compact**: cada palabra inicia con
    mayuscula, sin preposiciones, sin box-drawing.

    | Incorrecto | Correcto |
    |---|---|
    | `# ── DRY-RUN: generar reporte ──` | `# Dry-Run Generar Reporte` |
    | `# Primero inicializamos la variable` | `# Inicializa Variable` |
    | `# Esta función hace X porque...` | `# Calcula Hash SHA256` |

---

## Reglas De Trazabilidad (Headers/Footers)

17. **PROHIBIDO** escribir archivos .sh, .py, .yml, .yaml, .md, .cfg sin
    header/footer de desarrollo.

    TODO archivo debe INICIAR con:
    ```
    # Development By Angel Esquivel (CyberSecurity) [NEXUS CORE 2026]
    # =================================================================
    # Descripción Corta De Una Línea
    # =================================================================
    ```

    Y TERMINAR con:
    ```
    #End Development By Angel Esquivel (CyberSecurity) [NEXUS CORE 2026]
    ```

    Niveles de descripción:
    - Simple: 1 línea de descripción
    - Medio: 1 descripción + 1 detalle técnico
    - Complejo: 1 descripción + 3+ detalles

18. NO aplica a: .conf, .ini, .ldif (config de servicio, no código).

---

## Estandares De Código

### Idioma

| Contexto | Idioma | Ejemplo |
|---|---|---|
| Strings al usuario | espanol | `"Perfil invalido"` |
| Variables y funciones | ingles snake_case | `detect_resource_profile()` |
| Clases | ingles PascalCase | `ProjectOrchestrator` |
| Constantes | ingles UPPER_CASE | `MAX_RETRIES` |
| Comentarios | espanol PascalCase | `# Detecta Recursos Del Servidor` |

### Formato Python

- Type hints obligatorios
- Max 100 chars/línea
- 50 líneas max por función

### Formato Shell

- `set -euo pipefail` en entry points
- `local` para variables de función
- `readonly` para constantes
- `[[ ]]` en lugar de `[ ]` para condicionales
- Identacion: 4 espacios

### Formato Docker Compose

- Servicios ordenados por capa: proxy > db > cache > identity > apps
- Indentacion 2 espacios en YAML
- Nombres de servicio en snake_case
- Labels de logging via anchor `x-logging`

### Docstrings (Python)

```python
def process(input: str) -> dict:
    """Procesa datos y retorna resultado estructurado.

    Args:
        input: Datos de entrada.

    Returns:
        Diccionario con resultado.

    Raises:
        ValueError: Si input invalido.
    """
```

### Comentarios

```
# Correcto: Inicializa Variable De Configuración
# Correcto: Calcula Hash SHA256 Del Archivo

# Incorrecto: # Primero inicializamos la variable
# Incorrecto: # ── DRY-RUN: generar reporte ─────────
```

---

## Arquitectura

```
scripts/             <-- Automatización (backup, restore, validate, detect, wazuh)
config/              <-- Dockerfiles custom (privacyidea, odoo, openldap)
custom/privacyidea/  <-- Overlay 42 archivos (TOTP-only UI, license bypass)
projects/            <-- Proyectos cliente (1 por VPS)
docs/                <-- Documentación
data/                <-- Datos persistentes (gitignored)
```

### Capas De Red

| Capa | Red Docker | Servicios | Acceso Internet |
|---|---|---|---|
| Perimetral | docker_net | NPM, Authentik | Si (puertos 80/443) |
| Aplicaciones | docker_net | Odoo, Nextcloud, Vaultwarden, n8n, Kuma | Solo entrante |
| Datos | internal_net | PostgreSQL, Redis, OpenLDAP, PrivacyIDEA | NO |
| Host | N/A | Wazuh Agent, Trivy | Si (reportes SOC) |

---

## Orquestador De Proyectos

core-deploy.sh gestiona proyectos cliente con formato:

```
projects/core-<NN>-<cliente>-<DDMMYY>/
  .env              <-- Credenciales + branding + config por servicio
  branding/
    logo.png        <-- Logo del cliente para WebUI
  context.txt       <-- Metadatos del proyecto
```

### Flujo De Creacion

```
Menu: 1) Nuevo Proyecto
  |
  +-- Cliente (opcional, sanitizado a a-z0-9-)
  +-- Perfil 1-5 (3 intentos max, detecta recomendado por RAM)
  +-- Dominio (opcional)
  +-- SMTP (host, port, user, pass, from)
  +-- SSL (1=Let's Encrypt, 2=Cloudflare, 3 intentos max)
  +-- Resumen + Confirmar (s/n, 3 intentos max)
  +-- Desplegar ahora? (s/n, 3 intentos max)
```

### Acciones Por Proyecto

```
Menu: 2) Proyecto Existente
  +-- Lista projects/ con estado [activo|detenido]
  +-- Acciones: Desplegar, Validar, Backup, Estado, Logs, Destruir, Volver
```

---

## Patrones Obligatorios

### Subprocesos (Python, futuro)
```python
process = await asyncio.create_subprocess_exec(
    "cmd", "arg1",
    stdout=PIPE, stderr=PIPE
)
stdout, stderr = await asyncio.wait_for(
    process.communicate(), timeout=30
)
```

### Timeout I/O
```python
async with asyncio.timeout(10):
    response = await client.get(url)
```

### Error Handling
```python
return {
    "error": "mensaje claro",
    "tool": "nombre_tool",
    "code": "ERROR_CODE"
}
```

### Logging
```python
logger = logging.getLogger(__name__)
logger.info("Evento normal")
logger.error("Error operación")
# NUNCA: logger.info(f"API key: {key}")
```

---

## Variables De Entorno

Ver `.env.template` para lista completa.

| Grupo | Variables Requeridas |
|---|---|
| Proyecto | PROFILE, CLIENT_NAME, PROJECT_FOLDER |
| Branding | BRAND_PRIMARY_COLOR, BRAND_LOGO_PATH |
| Proxy | DOMAIN_BASE, SSL_MODE |
| DB | POSTGRES_PASSWORD, POSTGRES_USER |
| Redis | REDIS_PASSWORD |
| Authentik | AUTHENTIK_SECRET_KEY, AUTHENTIK_TOKEN |
| PrivacyIDEA | PI_ADMIN_PASSWORD, PI_SECRET_KEY, PI_PEPPER |
| Odoo | ODOO_ADMIN_PASSWD, ODOO_COMPANY_NAME |
| Vaultwarden | VW_ADMIN_TOKEN |
| LDAP | LDAP_ADMIN_PASSWORD, LDAP_CONFIG_PASSWORD |
| SMTP | SMTP_HOST, SMTP_USER, SMTP_PASSWORD |

---

## Docker Compose

```yaml
x-logging: &default-logging
  driver: "json-file"
  options:
    max-size: "20m"
    max-file: "3"

services:
  <nombre>:
    image: <imagen>:${VERSION:-default}
    container_name: <nombre>
    restart: unless-stopped
    security_opt: ["no-new-privileges:true"]
    healthcheck:
      test: [...]
      interval: 15s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          cpus: "${CPU_VAR:-0.50}"
          memory: "${MEM_VAR:-256M}"
```

---

## Documentación Relacionada

| Documento | Proposito |
|---|---|
| README.md | Vista general del proyecto |
| docs/AGENTS.md | Este archivo (fuente de verdad) |
| docs/TECHNICAL.md | Arquitectura técnica y redes |
| docs/SERVICES.md | Glosario detallado de cada servicio |
| docs/PROFILES.md | Tabla comparativa de 5 perfiles |
| docs/DEPLOYMENT.md | Guia paso a paso de despliegue |
| docs/CLOUDFLARE.md | Integración Cloudflare + Origin Cert |
| docs/CHANGELOG.md | Historial de versiones |
| docs/TEST_CYCLE.md | Pipeline de pruebas (12 fases P0-P11) |
| docs/RUNBOOK.md | Comandos exactos para cada fase del testing |

## Archivos Estándar

| Archivo | Crear si no existe |
|---|---|
| .gitignore | Siempre |
| .dockerignore | Siempre |
| README.md | Siempre |
| docs/CHANGELOG.md | Siempre |
| docs/AGENTS.md | Siempre |
| projects/.gitkeep | Siempre |

#End Development By Angel Esquivel (CyberSecurity) [NEXUS CORE 2026]
