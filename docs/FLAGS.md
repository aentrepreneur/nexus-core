<!-- Development By Angel Esquivel (Automation) [NEXUS CORE 2026] -->
# FLAGS — NEXUS CORE

## Contrato nuevo (`deploy.sh`)

| Flag | Tipo | Efecto |
|------|------|--------|
| `--dry-run` | global | verifica y muestra plan; no muta |
| `--json` | global | salida JSON en `check`/`status` |
| `--help` | global | ayuda |
| `--version` | global | version del contrato |
| `--` | global | separa argumentos del verbo |

## Verbos

| Verbo | Muta | Args | Notas |
|-------|------|------|-------|
| `check` | no | - | sintaxis, contrato, `.env`, imagenes |
| `status` | no | - | estado read-only |
| `deploy` | si | pasa a `core-deploy.sh` | menu de alta |
| `backup` | si | `--init-key`, `--files-only`, `--project`, `--retention` | cifrado |
| `restore` | si | `--latest`, `--dry-run`, `--yes`, `--no-db`, `--no-volumes` | verificado |
| `harden` | si | `--apply --yes`, `--disable-password-auth` | dry-run default |
| `images` | no | - | falla con `latest` |
| `bundle` | si | `--dry-run` | `dist/` |
| `sync` | si | `DEST`, `--apply`, `--dry-run` | rsync sin secretos |
| `menu` | no | - | paridad con `processes.conf` |

## Flags legacy de `core-deploy.sh`

Se conservan: `--new`, `--auto`, `--profile`, `--project`, `--deploy`,
`--list`, `--check`, `--validate`, `--destroy`, `--help`.

Pendiente: unificar `--check` (validacion) con `deploy.sh check` y eliminar
ambiguedades de `--project` + `--deploy` sin confirmacion.
<!-- End Development By Angel Esquivel (Automation) [NEXUS CORE 2026] -->
