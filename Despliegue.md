# 📘 Reporte Técnico: PokeDex Web App (Enterprise Edition)
## 🌐 Despliegue en la Nube con Azure Static Web Apps & CI/CD

### 📌 1. Descripción del Proyecto
La aplicación **PokeDex** es una plataforma robusta desarrollada bajo el framework **Angular**, diseñada para la gestión y visualización de datos Pokémon. 
Este proyecto documenta el ciclo de vida de despliegue en la nube utilizando **Microsoft Azure**, integrando flujos de trabajo automatizados (CI/CD) y una capa de seguridad endurecida mediante encabezados HTTP de grado profesional.

---

### ☁️ 2. Plataforma y Servicios Utilizados
| Servicio | Descripción |
| :--- | :--- |
| **Azure Static Web Apps** | Hosting optimizado para aplicaciones de una sola página (SPA). |
| **GitHub Actions** | Automatización de la compilación y despliegue continuo (CI/CD). |
| **Microsoft Oryx** | Motor de compilación automática que detecta el entorno (Node.js/Angular). |
| **SecurityHeaders** | Auditoría externa de seguridad para la validación de políticas HTTP. |

---

### 🚀 3. URL de la Aplicación
🔗 **Acceso público:** 👉 `https://ashy-sand-09d6db80f.5.azurestaticapps.net` 
*(Nota: Esta URL se deriva del nombre del secreto de implementación configurado)*

---

### 🔐 4. Seguridad de la Aplicación (Calificación A+)
Se implementó el archivo `staticwebapp.config.json` en la raíz del proyecto para mitigar vectores de ataque comunes.

#### 🛡️ Encabezados aplicados
| Encabezado | Función |
| :--- | :--- |
| **Content-Security-Policy** | Restringe la ejecución de scripts no autorizados (Anti-XSS). |
| **Strict-Transport-Security** | Garantiza que toda comunicación sea vía HTTPS (HSTS). |
| **X-Content-Type-Options** | Bloquea la interpretación incorrecta de tipos MIME (Anti-Sniffing). |
| **X-Frame-Options** | Establecido en `DENY` para evitar el secuestro de clics (Clickjacking). |
| **Referrer-Policy** | Limita la información enviada en las solicitudes de origen. |

---

### 🧰 5. Tecnologías Utilizadas
* **Framework:** Angular (TypeScript).
* **Gestor de paquetes:** NPM.
* **CI/CD:** GitHub Actions con `Azure/static-web-apps-deploy@v1`.
* **Infraestructura:** Azure Static Web Apps.

---

### 🧠 6. Reflexión Técnica y Resolución de Conflictos

#### ⚠️ Desafíos Críticos y Soluciones (Bitácora de Ingeniería)

1.  **Error de Autenticación (Unauthorized):**
    * **Problema:** El pipeline fallaba al intentar conectar con Azure debido a problemas de permisos.
    * **Resolución:** Se identificó un desajuste en el token de seguridad. Se actualizó el secreto en GitHub con el nombre `AZURE_STATIC_WEB_APPS_API_TOKEN_ASHY_SAND_09D6DB80F`.

2.  **Conflicto de Compilación (Error de carpeta `/dist`):**
    * **Problema:** El motor de build (Oryx) fallaba con el mensaje: `Failed to find a default file in the app artifacts folder (dist)`.
    * **Resolución:** Se detectó que Angular genera los archivos compilados en una subcarpeta. Se ajustó la configuración del pipeline para que Azure encontrara el archivo `index.html` en la ubicación correcta.

3.  **Estabilización del Pipeline:**
    * **Problema:** Se presentaron múltiples fallos consecutivos durante los intentos de actualización del código y configuración.
    * **Resolución:** Tras ajustar las rutas de salida y los secretos, se logró finalmente el estado **"Success" (Verde)** en el flujo de trabajo.

#### 📚 Aprendizaje obtenido
Este proyecto permitió comprender que la arquitectura de una aplicación (en este caso **Angular**) determina la configuración del pipeline de CI/CD. No basta con subir archivos; es necesario orquestar el proceso de construcción, gestionar secretos de nube y validar que las políticas de seguridad se apliquen correctamente sobre los artefactos generados.

---

### 👨‍💻 7. Autor
* **Nombre:** (Tu nombre aquí)
* **Curso:** (Tu curso aquí)
* **Estado Final:** **Despliegue Exitoso ✅**