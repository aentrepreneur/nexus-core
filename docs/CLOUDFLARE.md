# Integración Cloudflare para NEXUS CORE
# =================================================================

## Configuración Rápida

### 1. Agregar Dominio a Cloudflare

```
dash.cloudflare.com > Add a Domain
```

### 2. Generar Origin Certificate

```
SSL/TLS > Origin Server > Create Certificate
Hostnames: *.midominio.com y midominio.com
Validez: 15 anos
```

### 3. Importar en NPM Admin

```
Acceso: ssh -L 9090:localhost:81 user@servidor
Navegar: http://localhost:9090
SSL > Certificates > Add Custom Certificate
Pegar Origin Cert + Private Key
```

### 4. Configurar SSL en Cloudflare

```
SSL/TLS > Overview > Full (Strict)
```

### 5. Crear Registros DNS

```
Tipo  Nombre  Contenido            Proxy
A     npm     <IP-DEL-SERVIDOR>     Naranja
A     auth    <IP-DEL-SERVIDOR>     Naranja
A     odoo    <IP-DEL-SERVIDOR>     Naranja
... (por cada servicio)
```

### 6. Verificar

```bash
curl -sI https://npm.midominio.com | grep 'cf-ray'
# Debe mostrar cf-ray header (confirma paso por Cloudflare)
```

## Firewall: Solo Cloudflare

```bash
ufw default deny incoming
ufw allow 22/tcp
for ip in $(curl -s https://www.cloudflare.com/ips-v4); do
    ufw allow proto tcp from $ip to any port 80
    ufw allow proto tcp from $ip to any port 443
done
ufw enable
```

## Ver docs/CLOUDFLARE.md en npm-2026 para guia completa
## (aplica el mismo proceso para NEXUS CORE)

#End Development By Angel Esquivel (CyberSecurity) [NEXUS CORE 2026]
