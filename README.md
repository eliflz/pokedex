# PokeDex Web App

> Aplicación Angular desplegada en Microsoft Azure Static Web Apps con pipeline CI/CD y cabeceras de seguridad configuradas.

---

## 📌 Descripción del Proyecto

**PokeDex** es una aplicación web de página única (SPA) desarrollada con **Angular 14**, diseñada para la consulta y visualización de datos Pokémon. Consume la API pública de [PokéAPI](https://pokeapi.co) mediante GraphQL y ofrece una interfaz con filtros por tipo, generación y estadísticas.

El proyecto fue desplegado en la nube mediante **Azure Static Web Apps**, integrando un pipeline de CI/CD automatizado con **GitHub Actions** y configurando cabeceras de seguridad HTTP que permiten obtener una calificación **A+** en [SecurityHeaders.com](https://securityheaders.com).

---

## 🌐 URL Pública

**🔗 [https://ashy-sand-09d6db80f.2.azurestaticapps.net](https://ashy-sand-09d6db80f.2.azurestaticapps.net)**

---

## 🛠️ Tecnologías Utilizadas

| Tecnología | Descripción |
|:---|:---|
| **Angular 14** | Framework frontend (SPA) |
| **TypeScript** | Lenguaje principal |
| **GraphQL / Apollo** | Consumo de la PokéAPI |
| **SCSS** | Estilos de la aplicación |
| **Azure Static Web Apps** | Hosting en la nube |
| **GitHub Actions** | Pipeline de CI/CD |
| **Microsoft Oryx** | Motor de compilación automática |
| **SecurityHeaders.com** | Auditoría de seguridad HTTP |

---

## ⚙️ Requisitos Previos (Instalación Local)

Antes de correr el proyecto localmente necesitas tener instalado:

- **Node.js** v16 o superior
- **npm** v8 o superior
- **Angular CLI** v14

```bash
# Instalar Angular CLI globalmente
npm install -g @angular/cli@14
```

---

## 🚀 Instalación y Ejecución Local

```bash
# 1. Clonar el repositorio
git clone https://github.com/eliflz/pokedex-main.git
cd pokedex-main

# 2. Instalar dependencias
npm install

# 3. Ejecutar en modo desarrollo
ng serve

# La app quedará disponible en http://localhost:4200
```

### Compilación para producción

```bash
ng build --configuration production
# Los archivos compilados quedan en /dist/pokedex-angular
```

---

## ☁️ Despliegue en Azure Static Web Apps

### Paso 1 — Creación de cuenta en Azure for Students

Accedí a [https://azure.microsoft.com/es-es/free/students](https://azure.microsoft.com/es-es/free/students) y activé la suscripción **Azure for Students** con mi correo institucional, obteniendo **$100 USD** en créditos gratuitos sin necesidad de tarjeta de crédito.

> **Evidencia:** Se validó el acceso con suscripción activa desde el [Portal de Azure](https://portal.azure.com).

---

### Paso 2 — Creación del recurso Static Web Apps

Desde el portal de Azure:

1. Busqué el servicio **"Static Web Apps"** en la barra de búsqueda.
2. Hice clic en **"Crear"** y configuré:
   - **Suscripción:** Azure for Students
   - **Grupo de recursos:** `rg-pokedex`
   - **Nombre del recurso:** `pokedex-app`
   - **Plan de hospedaje:** Free
   - **Región:** East US 2
3. En la sección de origen vinculé mi **repositorio de GitHub**.
4. Confirmé con **"Revisar y crear"** → **"Crear"**.

Azure generó automáticamente el archivo `.github/workflows/pipelines.yml` en el repositorio.

---

### Paso 3 — Variables de entorno y secretos de construcción

Azure provee un **token de implementación** para autenticar el pipeline de GitHub Actions. Lo registré como secreto en el repositorio:

1. En Azure: **Recurso → "Administrar token de implementación"** → copié el token.
2. En GitHub: **Settings → Secrets and variables → Actions → New repository secret**
3. Nombre del secreto: `AZURE_STATIC_WEB_APPS_API_TOKEN`

Este secreto es referenciado en el archivo de workflow:

```yaml
# .github/workflows/pipelines.yml (fragmento)
- name: Build And Deploy
  uses: Azure/static-web-apps-deploy@v1
  with:
    azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
    repo_token: ${{ secrets.GITHUB_TOKEN }}
    action: "upload"
    app_location: "/"
    output_location: "dist/pokedex-angular"
```

> ⚙️ **Configuración en la nube:** Las variables de entorno y el token de construcción se configuraron directamente en la sección **"Configuración"** del recurso Azure Static Web Apps, accesible desde el Portal de Azure bajo el recurso `pokedex-app`.

---

### Paso 4 — Ejecución del pipeline CI/CD

Cada `push` a la rama `main` dispara automáticamente el pipeline:

1. **Checkout** del código.
2. **Build** con Node.js 16 y `ng build --configuration production`.
3. **Deploy** hacia Azure Static Web Apps.

El log del paso **"Build And Deploy"** confirma la URL pública del despliegue.

---

## 🔐 Seguridad — Cabeceras HTTP Configuradas

Las cabeceras de seguridad se configuraron en el archivo **`staticwebapp.config.json`** en la raíz del proyecto. Esto indica a Azure qué headers HTTP debe incluir en cada respuesta.

```json
{
  "globalHeaders": {
    "Content-Security-Policy": "default-src 'self'; script-src 'self' 'sha256-...'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com; img-src 'self' data: https://raw.githubusercontent.com https://assets.pokemon.com; connect-src 'self' https://pokeapi.co https://beta.pokeapi.co;",
    "Strict-Transport-Security": "max-age=31536000; includeSubDomains; preload",
    "X-Content-Type-Options": "nosniff",
    "X-Frame-Options": "DENY",
    "Referrer-Policy": "no-referrer",
    "Permissions-Policy": "geolocation=(), microphone=(), camera=(), payment=(), usb=(), interest-cohort=()",
    "X-Permitted-Cross-Domain-Policies": "none",
    "Cross-Origin-Opener-Policy": "same-origin",
    "Cross-Origin-Resource-Policy": "same-origin"
  }
}
```

### Explicación de cada cabecera

| Header | Función |
|:---|:---|
| `Content-Security-Policy` | Define qué orígenes pueden cargar recursos (scripts, estilos, imágenes, conexiones). Previene XSS. |
| `Strict-Transport-Security` | Fuerza HTTPS durante 1 año e incluye subdominios y preload. Previene ataques de degradación. |
| `X-Content-Type-Options: nosniff` | Impide que el navegador adivine el tipo MIME. Previene ataques de MIME sniffing. |
| `X-Frame-Options: DENY` | Prohíbe que la app sea embebida en iframes. Previene clickjacking. |
| `Referrer-Policy: no-referrer` | No envía información del origen al navegar a otros sitios. Protege la privacidad. |
| `Permissions-Policy` | Deshabilita APIs del navegador no usadas (cámara, micrófono, geolocalización, pagos). |
| `X-Permitted-Cross-Domain-Policies: none` | Bloquea el acceso de plugins Flash/Acrobat a datos del dominio. |
| `Cross-Origin-Opener-Policy` | Aísla el contexto de navegación para proteger contra ataques cross-origin. |
| `Cross-Origin-Resource-Policy` | Restringe que otros orígenes carguen los recursos de este sitio. |

### Resultado en SecurityHeaders.com

> 🏆 **Puntuación obtenida: A+**
>
> Verificado en: [https://securityheaders.com/?q=https://ashy-sand-09d6db80f.2.azurestaticapps.net](https://securityheaders.com/?q=https://ashy-sand-09d6db80f.2.azurestaticapps.net)

---

## 📁 Estructura del Proyecto

```
pokedex-main/
├── src/
│   ├── app/
│   │   ├── core/          # Servicios, interceptores, guards
│   │   ├── data/          # GraphQL queries, enums, mocks
│   │   ├── shared/        # Componentes, pipes, directivas reutilizables
│   │   └── views/         # Páginas: home, pokemon-details, about, 404, 500
│   ├── assets/            # Imágenes, fuentes, estilos globales
│   └── environments/      # Configuración de entornos (dev/prod)
├── documentacion/
│   └── Despliegue.md      # Documentación detallada del despliegue
├── staticwebapp.config.json  # Cabeceras de seguridad HTTP
├── .github/workflows/
│   └── pipelines.yml      # Pipeline CI/CD de GitHub Actions
├── angular.json
├── package.json
└── README.md
```

---

## 📄 Documentación Adicional

Consulta el archivo [`documentacion/Despliegue.md`](documentacion/Despliegue.md) para ver el proceso detallado de configuración, los errores encontrados durante el despliegue y las soluciones aplicadas.

---

## 👨‍💻 Autor

- **Nombre:** Eliecer Arias Flórez
- **Curso:** Sistemas Distribuidos — 1598
- **Institución:** Fundación Universitaria Tecnológico Comfenalco
