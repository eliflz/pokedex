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

**Acceso público:** `https://ashy-sand-09d6db80f.5.azurestaticapps.net`

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

Azure proporciona un token de autenticación que debe registrarse como secreto en el repositorio para que el pipeline tenga permisos de despliegue.

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

El archivo generado inicialmente presentó errores que impidieron el despliegue correcto. Fue necesario modificarlo manualmente para que funcionara con la estructura de Angular.

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
          output_location: "dist/pokedex-angular"   # <- ruta corregida manualmente

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

---

### 4.4 — Configuración de seguridad (`staticwebapp.config.json`)

Se creó el archivo `staticwebapp.config.json` en la raíz del proyecto para implementar encabezados HTTP de seguridad:

```json
{
  "globalHeaders": {
    "Content-Security-Policy": "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https://raw.githubusercontent.com; connect-src 'self' https://pokeapi.co https://beta.pokeapi.co;",
    "Strict-Transport-Security": "max-age=31536000; includeSubDomains",
    "X-Content-Type-Options": "nosniff",
    "X-Frame-Options": "DENY",
    "Referrer-Policy": "no-referrer"
  },
  "navigationFallback": {
    "rewrite": "/index.html",
    "exclude": ["/assets/*", "/*.{css,js,png,gif,ico,svg}"]
  }
}
```

| Encabezado | Función |
|:---|:---|
| **Content-Security-Policy** | Restringe recursos externos; permite explícitamente la PokeAPI. |
| **Strict-Transport-Security** | Garantiza que toda comunicación sea vía HTTPS (HSTS). |
| **X-Content-Type-Options** | Bloquea la interpretación incorrecta de tipos MIME. |
| **X-Frame-Options** | Evita el secuestro de clics (Clickjacking). |
| **Referrer-Policy** | Limita la información enviada en solicitudes de origen cruzado. |

---

### 4.5 — Verificación del despliegue en GitHub Actions

Una vez aplicados los cambios y realizado el `push` a la rama `main`:

1. Ir al repositorio → pestaña **Actions**.
2. Seleccionar el flujo de trabajo más reciente: `Azure Static Web Apps CI/CD`.
3. Confirmar que el job `Build and Deploy Job` muestra estado **Success** (indicador verde ✅).
4. Acceder a la URL pública para verificar que la aplicación carga correctamente:
   `https://ashy-sand-09d6db80f.5.azurestaticapps.net`

> La verificación en GitHub Actions fue el paso definitivo para confirmar que el pipeline funcionaba correctamente de extremo a extremo.

---

## 5. Errores Encontrados y Soluciones

### Error 1 — Fallo de compilación en el YML (`dist` folder not found)

**Síntoma:** El motor Oryx lanzaba el siguiente error en GitHub Actions:

```
Failed to find a default file in the app artifacts folder (dist)
```

**Causa:** Angular no genera los archivos compilados directamente en `/dist`, sino en una subcarpeta con el nombre exacto del proyecto: `/dist/pokedex-angular`. El archivo YML generado automáticamente por Azure apuntaba a la ruta genérica `/dist`, que no existe tras la compilación.

**Solución:** Modificar manualmente el campo `output_location` en el archivo YML:

```yaml
# Antes (incorrecto — generado automáticamente por Azure):
output_location: "dist"

# Después (correcto — corregido manualmente):
output_location: "dist/pokedex-angular"
```

Una vez aplicado este cambio y realizado el push, el pipeline compiló correctamente y alcanzó el estado **Success** en GitHub Actions.

---

### Error 2 — Error 500 en la aplicación desplegada (PokeAPI bloqueada por CSP)

**Síntoma:** La aplicación cargaba en Azure pero al intentar consultar cualquier Pokémon aparecía la pantalla de error 500 con el mensaje:

```
What? POKÉAPI is evolving?
We're not sure. The truth is that it didn't respond what we expected...
```

**Causa 1 — Content-Security-Policy demasiado restrictivo:**
El archivo `staticwebapp.config.json` tenía configurado `default-src 'self'`, lo que le indica al navegador que solo puede conectarse al propio dominio de Azure. Cuando Angular intentaba hacer peticiones a `https://pokeapi.co` y `https://beta.pokeapi.co`, el navegador las bloqueaba silenciosamente. El interceptor HTTP de la aplicación (`request.interceptor.ts`) capturaba ese fallo de red y redirigía automáticamente a la ruta `/error`, mostrando la pantalla de error 500.

**Causa 2 — Ruta de imágenes incorrecta en producción:**
El archivo `src/environments/environment.prod.ts` tenía configurada la ruta de imágenes como `/pokedex-angular/assets/images`, que corresponde a un despliegue en subdirectorio (por ejemplo, GitHub Pages). En Azure Static Web Apps la aplicación se despliega en la raíz del dominio, por lo que esa ruta no existía y las imágenes no cargaban.

**Solución — Corrección del `staticwebapp.config.json`:**

Se añadió la directiva `connect-src` al CSP para permitir explícitamente las llamadas a la PokeAPI:

```json
{
  "globalHeaders": {
    "Content-Security-Policy": "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https://raw.githubusercontent.com; connect-src 'self' https://pokeapi.co https://beta.pokeapi.co;",
    "Strict-Transport-Security": "max-age=31536000; includeSubDomains",
    "X-Content-Type-Options": "nosniff",
    "X-Frame-Options": "DENY",
    "Referrer-Policy": "no-referrer"
  },
  "navigationFallback": {
    "rewrite": "/index.html",
    "exclude": ["/assets/*", "/*.{css,js,png,gif,ico,svg}"]
  }
}
```

**Solución — Corrección del `environment.prod.ts`:**

Se corrigió la ruta de imágenes eliminando el prefijo `/pokedex-angular`:

```typescript
// Antes (incorrecto — ruta para subdirectorio):
imagesPath: '/pokedex-angular/assets/images',

// Después (correcto — ruta para raíz del dominio en Azure):
imagesPath: '/assets/images',
```

Tras aplicar ambas correcciones y hacer push, la aplicación cargó correctamente sin errores.

---

## 6. Reflexión Técnica

Este proyecto permitió comprender que el despliegue de una aplicación Angular en la nube involucra mucho más que subir archivos. Es necesario:

- Conocer la **estructura de salida del framework** (`dist/pokedex-angular`) para configurar correctamente el pipeline.
- Entender cómo funciona la **Content-Security-Policy** y qué dominios externos debe autorizar explícitamente la aplicación.
- Distinguir entre **entornos de despliegue** (subdirectorio vs. raíz) para configurar rutas de assets correctamente.
- Gestionar correctamente los **secretos de autenticación** entre GitHub y Azure.
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
