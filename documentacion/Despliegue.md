#  PokeDex Web App 
### Despliegue en la Nube con Azure Static Web Apps 

---

##  Descripción del Proyecto

Desarrollé **PokeDex**, una plataforma robusta bajo el framework **Angular**, diseñada para la gestión y visualización de datos Pokémon.

En este .MD documento el ciclo de vida completo del despliegue en la nube utilizando **Microsoft Azure**, incluyendo la configuración del pipeline CI/CD, los errores que encontré durante el proceso y las soluciones que apliqué para lograr un despliegue exitoso.

---

## ⚙️ Plataforma y Servicios que Utilicé

| Servicio | Descripción |
|:---|:---|
| **Azure Static Web Apps** | Hosting optimizado para aplicaciones de una sola página (SPA). |
| **GitHub Actions** | Automatización de la compilación y despliegue continuo (CI/CD). |
| **Microsoft Oryx** | Motor de compilación automática que detecta el entorno (Node.js/Angular). |
| **SecurityHeaders** | Auditoría externa de seguridad para la validación de políticas HTTP. |

---

## 🌐 URL de la Aplicación

**Acceso público:** [https://ashy-sand-09d6db80f.2.azurestaticapps.net](https://ashy-sand-09d6db80f.2.azurestaticapps.net)

> Confirmé esta URL directamente en el log del paso **"Build And Deploy"** de GitHub Actions.

---

##  Proceso de Despliegue Paso a Paso

### Paso 1 — Creé el recurso en Azure

1. Accedí al Portal de Azure: [https://portal.azure.com](https://portal.azure.com)
2. Busqué el servicio **"Static Web Apps"** en la barra de búsqueda.
3. Hice clic en **"Crear"** y completé los campos:
   - **Suscripción:** Free Trial
   - **Grupo de recursos:** `rg-pokedex`
   - **Nombre:** `pokedex-app`
   - **Plan de hospedaje:** Free
   - **Región:** East US 2
4. En la sección de despliegue, seleccioné **GitHub** como origen y autoricé el acceso al repositorio.
5. Hice clic en **"Revisar y crear"** → **"Crear"**.

> Azure generó automáticamente un archivo `.github/workflows/azure-static-web-apps-*.yml` en mi repositorio al vincular la cuenta.

---

### Paso 2 — Configuré el secreto en GitHub

Azure me proporcionó un token de autenticación que registré como secreto en el repositorio para que el pipeline tuviera permisos de despliegue.

**Pasos que seguí:**
1. En el Portal de Azure, fui al recurso creado → **"Administrar token de implementación"**.
2. Copié el token generado.
3. En GitHub, fui a **Settings → Secrets and variables → Actions → New repository secret**.
4. Configuré el secreto con el nombre exacto:

```
AZURE_STATIC_WEB_APPS_API_TOKEN_ASHY_SAND_09D6DB80F
```

5. Pegué el token copiado y guardé.

---

### Paso 3 — Configuré el archivo YML (Pipeline CI/CD)

Azure generó automáticamente un archivo de flujo de trabajo en la ruta:

```
.github/workflows/azure-static-web-apps-ashy-sand-09d6db80f.yml
```

El archivo generado inicialmente presentó múltiples errores que tuve que corregir manualmente en varias iteraciones. El archivo final funcional quedó así:

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
          lfs: false

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm install

      - name: Build Angular app
        run: npx ng build --configuration production

      - name: Build And Deploy
        id: builddeploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_ASHY_SAND_09D6DB80F }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: "upload"
          app_location: "dist/pokedex-angular"   # <- apunta directo al build
          api_location: ""
          output_location: ""                     # <- vacío porque ya apuntamos al dist
          config_file_location: "/"
          skip_app_build: true

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

### Paso 4 — Configuré la seguridad (`staticwebapp.config.json`)

Creé el archivo `staticwebapp.config.json` en la raíz del proyecto para implementar encabezados HTTP de seguridad:

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
| **Content-Security-Policy** | Restringe recursos externos; permití explícitamente la PokeAPI. |
| **Strict-Transport-Security** | Garantiza que toda comunicación sea vía HTTPS (HSTS). |
| **X-Content-Type-Options** | Bloquea la interpretación incorrecta de tipos MIME. |
| **X-Frame-Options** | Evita el secuestro de clics (Clickjacking). |
| **Referrer-Policy** | Limita la información enviada en solicitudes de origen cruzado. |

---

### Paso 5 — Verifiqué el despliegue en GitHub Actions

Una vez que apliqué todos los cambios y realicé el `push` a la rama `main`:

1. Fui al repositorio → pestaña **Actions**.
2. Seleccioné el flujo de trabajo más reciente: `Azure Static Web Apps CI/CD`.
3. Expandí el paso **"Build And Deploy"** y confirmé la línea:
```
   Status: Succeeded.
   Deployment Complete :)
   Visit your site at: https://ashy-sand-09d6db80f.2.azurestaticapps.net
```
4. Accedí a la URL pública y verifiqué que la aplicación cargaba correctamente.

---

## 🐛 Errores que Encontré y Cómo los Resolví

### Error 1 — Fallo de compilación en el YML (`dist` folder not found)

**Síntoma:** El motor Oryx lanzaba el siguiente error en GitHub Actions:

```
Failed to find a default file in the app artifacts folder (dist)
```

**Causa:** Angular no genera los archivos compilados directamente en `/dist`, sino en una subcarpeta con el nombre exacto del proyecto: `/dist/pokedex-angular`. El archivo YML que Azure generó automáticamente apuntaba a la ruta genérica `dist`, que no existe tras la compilación.

**Solución que apliqué:**

```yaml
# Antes (incorrecto — generado por Azure):
output_location: "dist"

# Después (correcto — corregido manualmente):
output_location: "dist/pokedex-angular"
```

---

### Error 2 — Error 500 en la aplicación desplegada (PokeAPI bloqueada por CSP)

**Síntoma:** La aplicación cargaba en Azure pero al intentar consultar cualquier Pokémon aparecía la pantalla de error 500 con el mensaje:

```
What? POKÉAPI is evolving?
We're not sure. The truth is that it didn't respond what we expected...
```

**Causa 1 — Content-Security-Policy demasiado restrictivo:**
Tenía `default-src 'self'` en el `staticwebapp.config.json`, lo que bloqueaba silenciosamente todas las peticiones HTTP a dominios externos. El interceptor HTTP de Angular capturaba ese fallo y redirigía a la ruta `/error`.

**Causa 2 — Ruta de imágenes incorrecta en producción:**
El archivo `environment.prod.ts` tenía la ruta `/pokedex-angular/assets/images`, válida para GitHub Pages pero no para Azure, donde la app vive en la raíz del dominio.

**Solución en `staticwebapp.config.json`:**

```json
"connect-src 'self' https://pokeapi.co https://beta.pokeapi.co;"
```

**Solución en `environment.prod.ts`:**

```typescript
// Antes:
imagesPath: '/pokedex-angular/assets/images',

// Después:
imagesPath: '/assets/images',
```

---

### Error 3 — Error 404 en la URL raíz tras despliegue exitoso

**Síntoma:** GitHub Actions marcaba el pipeline en verde ✅ pero la URL pública devolvía error 404.

**Causa:** El proyecto usa `@angular-builders/custom-webpack` con un `webpack.config.ts` personalizado. El motor Oryx de Azure no sabe manejar este builder y generaba la carpeta `dist` vacía, aunque el paso de despliegue no fallara explícitamente.

**Iteración fallida:** Intenté usar `skip_app_build: true` manteniendo `app_location: "/"` y `output_location: "dist/pokedex-angular"`. Azure ignoró el `output_location` y buscó el `index.html` en la raíz, lanzando:

```
Failed to find a default file in the app artifacts folder (/).
```

**Solución definitiva que funcionó:** Compilar manualmente con Node.js en el pipeline y apuntar `app_location` directamente al directorio del build ya generado:

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v3
  with:
    node-version: '18'

- name: Install dependencies
  run: npm install

- name: Build Angular app
  run: npx ng build --configuration production

- name: Build And Deploy
  uses: Azure/static-web-apps-deploy@v1
  with:
    app_location: "dist/pokedex-angular"  # apunta al build ya generado
    output_location: ""                    # vacío, no hay compilación adicional
    skip_app_build: true                   # Azure no recompila
```

---

### Error 4 — URL incorrecta en la documentación

**Síntoma:** Accedía a `https://ashy-sand-09d6db80f.5.azurestaticapps.net` y daba error 404 o "sitio no disponible".

**Causa:** Asumí la URL incorrectamente. Azure asigna un número de entorno en la URL que solo se puede confirmar leyendo el log del paso "Build And Deploy" en GitHub Actions.

**Solución:** Leí el log del pipeline hasta encontrar la línea:

```
Visit your site at: https://ashy-sand-09d6db80f.2.azurestaticapps.net
```

La URL correcta y confirmada por Azure es la que termina en `.2.azurestaticapps.net`.

---

## 💡 Lo que Aprendí

Este proyecto me demostró que el despliegue de una aplicación Angular en la nube involucra mucho más que subir archivos. Las lecciones clave que me llevé fueron:

- **El builder importa:** `@angular-builders/custom-webpack` impide que Oryx compile correctamente. Tuve que tomar el control del build manualmente en el pipeline.
- **`skip_app_build` cambia el comportamiento de `output_location`:** Cuando está activo, Azure ignora `output_location` y busca en `app_location`. La solución es apuntar `app_location` al directorio del build.
- **La CSP bloquea APIs externas silenciosamente:** Un `default-src 'self'` sin `connect-src` explícito impide todas las llamadas HTTP a dominios externos.
- **Los entornos de despliegue difieren:** Una ruta válida en GitHub Pages no lo es en Azure Static Web Apps.
- **La URL real solo la confirma el log de Azure:** No asumir la URL — siempre verificarla en la línea `Visit your site at:` del log de GitHub Actions.

---

## 👤 Autor

| Campo | Detalle |
|:---|:---|
| **Nombre** | Eliecer Arias Flórez |
| **Curso** | Sistemas Distribuidos — 1598 |
| **Institución** | Fundación Universitaria Tecnológico Comfenalco |
| **Estado final** | Despliegue exitoso ✅ |
