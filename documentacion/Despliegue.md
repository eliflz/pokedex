# Reporte Técnico: PokeDex Web App — Enterprise Edition
## Despliegue en la Nube con Azure Static Web Apps & CI/CD

---

## 1. Descripción del Proyecto

La aplicación **PokeDex** es una plataforma robusta desarrollada bajo el framework **Angular**, diseñada para la gestión y visualización de datos Pokémon.

Este documento detalla el ciclo de vida completo del despliegue en la nube utilizando **Microsoft Azure**, incluyendo la configuración del pipeline CI/CD, los errores encontrados durante el proceso y las soluciones aplicadas para lograr un despliegue exitoso.

---

## 2. Plataforma y Servicios Utilizados

| Servicio | Descripción |
|:---|:---|
| **Azure Static Web Apps** | Hosting optimizado para aplicaciones de una sola página (SPA). |
| **GitHub Actions** | Automatización de la compilación y despliegue continuo (CI/CD). |
| **Microsoft Oryx** | Motor de compilación automática que detecta el entorno (Node.js/Angular). |
| **SecurityHeaders** | Auditoría externa de seguridad para la validación de políticas HTTP. |

---

## 3. URL de la Aplicación

**Acceso público:** `[https://ashy-sand-09d6db80f.5.azurestaticapps.net](https://ashy-sand-09d6db80f.2.azurestaticapps.net/error)`

> Esta URL se deriva del nombre del secreto de implementación configurado en el repositorio de GitHub.

---

## 4. Proceso de Despliegue Paso a Paso

### 4.1 — Creación del recurso en Azure

1. Acceder al Portal de Azure: [https://portal.azure.com](https://portal.azure.com)
2. Buscar el servicio **"Static Web Apps"** en la barra de búsqueda.
3. Hacer clic en **"Crear"** y completar los campos:
   - **Suscripción:** Free Trial
   - **Grupo de recursos:** `rg-pokedex`
   - **Nombre:** `pokedex-app`
   - **Plan de hospedaje:** Free
   - **Región:** East US 2
4. En la sección de despliegue, seleccionar **GitHub** como origen y autorizar el acceso al repositorio.
5. Hacer clic en **"Revisar y crear"** → **"Crear"**.

> Azure genera automáticamente un archivo `.github/workflows/azure-static-web-apps-*.yml` en el repositorio al vincular la cuenta.

---

### 4.2 — Configuración del secreto en GitHub

Azure proporciona un token de autenticación (`API Token`) que debe registrarse como secreto en el repositorio para que el pipeline tenga permisos de despliegue.

**Pasos:**
1. En el Portal de Azure, ir al recurso creado → **"Administrar token de implementación"**.
2. Copiar el token generado.
3. En GitHub, ir a **Settings → Secrets and variables → Actions → New repository secret**.
4. Configurar el secreto con el nombre exacto:

```
AZURE_STATIC_WEB_APPS_API_TOKEN_ASHY_SAND_09D6DB80F
```

5. Pegar el token copiado y guardar.

---

### 4.3 — Configuración del archivo YML (Pipeline CI/CD)

Azure genera automáticamente un archivo de flujo de trabajo en la ruta:

```
.github/workflows/azure-static-web-apps-ashy-sand-09d6db80f.yml
```

**El archivo generado inicialmente presentó errores** que impidieron el despliegue correcto. Fue necesario modificarlo manualmente para que funcionara con la estructura de Angular.

**Archivo YML corregido y funcional:**

```yaml
name: Azure Static Web Apps CI/CD

on:
  push:
    branches:
      - main
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches:
      - main

jobs:
  build_and_deploy_job:
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
    runs-on: ubuntu-latest
    name: Build and Deploy Job

    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true

      - name: Build And Deploy
        id: builddeploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_ASHY_SAND_09D6DB80F }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: "upload"
          app_location: "/"
          api_location: ""
          output_location: "dist/pokedex"   # <- ruta corregida manualmente

  close_pull_request_job:
    if: github.event_name == 'pull_request' && github.event.action == 'closed'
    runs-on: ubuntu-latest
    name: Close Pull Request Job

    steps:
      - name: Close Pull Request
        id: closepullrequest
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_ASHY_SAND_09D6DB80F }}
          action: "close"
```

> El cambio clave fue ajustar `output_location` de `"dist"` a `"dist/pokedex"`, que es la subcarpeta donde Angular genera los archivos compilados.

---

### 4.4 — Configuración de seguridad (`staticwebapp.config.json`)

Se creó el archivo `staticwebapp.config.json` en la raíz del proyecto para implementar encabezados HTTP de seguridad:

```json
{
  "globalHeaders": {
    "Content-Security-Policy": "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline';",
    "Strict-Transport-Security": "max-age=31536000; includeSubDomains",
    "X-Content-Type-Options": "nosniff",
    "X-Frame-Options": "DENY",
    "Referrer-Policy": "strict-origin-when-cross-origin"
  }
}
```

| Encabezado | Función |
|:---|:---|
| **Content-Security-Policy** | Restringe la ejecución de scripts no autorizados (Anti-XSS). |
| **Strict-Transport-Security** | Garantiza que toda comunicación sea vía HTTPS (HSTS). |
| **X-Content-Type-Options** | Bloquea la interpretación incorrecta de tipos MIME (Anti-Sniffing). |
| **X-Frame-Options** | Evita el secuestro de clics (Clickjacking). |
| **Referrer-Policy** | Limita la información enviada en solicitudes de origen cruzado. |

---

### 4.5 — Verificación del despliegue en GitHub Actions

Una vez aplicados los cambios y realizado el `push` a la rama `main`, se verificó el resultado directamente en la pestaña **Actions** del repositorio de GitHub:

1. Ir al repositorio → pestaña **Actions**.
2. Seleccionar el flujo de trabajo más reciente: `Azure Static Web Apps CI/CD`.
3. Confirmar que el job `Build and Deploy Job` muestra estado **Success** (indicador verde ✅).
4. Acceder a la URL pública para verificar que la aplicación carga correctamente:
   `https://ashy-sand-09d6db80f.5.azurestaticapps.net`

> La verificación en GitHub Actions fue el paso definitivo para confirmar que el pipeline estaba funcionando correctamente de extremo a extremo.

---

## 5. Errores Encontrados y Soluciones

### Error 1 — Autenticación fallida (Unauthorized)

**Síntoma:** El pipeline fallaba en el paso de despliegue con un error de autenticación.

**Causa:** El nombre del secreto en GitHub no coincidía con el referenciado en el archivo YML, o el token había sido regenerado en Azure sin actualizar el secreto.

**Solución:**
- Regenerar el token desde el Portal de Azure → recurso Static Web Apps → **"Administrar token de implementación"**.
- Eliminar el secreto anterior en GitHub y crear uno nuevo con el nombre exacto:
```
  AZURE_STATIC_WEB_APPS_API_TOKEN_ASHY_SAND_09D6DB80F
```

---

### Error 2 — Fallo de compilación (`dist` folder not found)

**Síntoma:** El motor Oryx lanzaba el error:
```
Failed to find a default file in the app artifacts folder (dist)
```

**Causa:** Angular no genera los archivos compilados directamente en `/dist`, sino en una subcarpeta con el nombre del proyecto: `/dist/pokedex`. El archivo YML generado automáticamente por Azure apuntaba a la ruta incorrecta.

**Solución:** Modificar manualmente el campo `output_location` en el archivo YML:
```yaml
# Antes (incorrecto):
output_location: "dist"

# Después (correcto):
output_location: "dist/pokedex"
```

---

### Error 3 — Fallos consecutivos tras actualizaciones

**Síntoma:** Múltiples ejecuciones del pipeline marcadas como fallidas en GitHub Actions al intentar aplicar correcciones incrementales.

**Causa:** Combinación de los errores anteriores sin que ninguno estuviera resuelto de forma completa al momento de cada push.

**Solución:** Resolver primero el error del secreto (Error 1) y luego el de la ruta de compilación (Error 2) antes de hacer un nuevo push. Una vez aplicados ambos cambios de forma conjunta, el pipeline alcanzó el estado **Success** de forma estable.

---

## 6. Reflexión Técnica

Este proyecto permitió comprender que la arquitectura de una aplicación determina directamente la configuración del pipeline de CI/CD. No basta con vincular un repositorio a Azure; es necesario:

- Conocer la **estructura de salida del framework** (en Angular, `dist/nombre-proyecto`).
- Gestionar correctamente los **secretos de autenticación** entre GitHub y Azure.
- Validar que las **políticas de seguridad HTTP** se apliquen sobre los artefactos generados.
- Verificar el resultado final tanto en **GitHub Actions** como en la **URL pública** de la aplicación.

El indicador verde en GitHub Actions no es solo una confirmación técnica; es la evidencia de que todo el ciclo — código, build, seguridad y despliegue — funciona de forma orquestada y automatizada.

---

## 7. Autor

| Campo | Detalle |
|:---|:---|
| **Nombre** | (Tu nombre aquí) |
| **Curso** | (Tu curso aquí) |
| **Institución** | (Tu centro educativo) |
| **Estado final** | Despliegue exitoso ✅ |

---

*Documento técnico generado como parte del proyecto PokeDex Web App — Pueblo Paleta Inc.*
