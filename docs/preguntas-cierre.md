# Preguntas de cierre - EC1 F1 A1

## 1. ¿Qué función cumple Node.js en el entorno de desarrollo de una aplicación frontend?
Node.js actúa como el entorno de ejecución (*runtime*) de JavaScript en el lado del servidor y en la máquina local. Aunque el código final de la aplicación frontend se ejecuta en el navegador del usuario, Node.js es indispensable durante la fase de desarrollo para ejecutar herramientas de automatización, gestores de paquetes (pnpm/npm) y servidores de desarrollo locales como Vite.

---

## 2. ¿Qué es PNPM y qué responsabilidad tiene dentro del proyecto?
PNPM es un gestor de paquetes (*Package Manager*) para Node.js. Su responsabilidad principal es descargar, instalar, actualizar y administrar todas las dependencias y librerías externas que requiere el proyecto. Destaca por ser altamente eficiente en el uso del espacio en disco y tiempo de instalación gracias a su arquitectura basada en un almacén global (*global store*) con enlaces simbólicos (*symlinks*).

---

## 3. ¿Qué problema resuelve Vite durante el desarrollo?
Vite resuelve los lentos tiempos de inicio y recarga (*Hot Module Replacement* / HMR) que sufren los empaquetadores tradicionales (como Webpack o Rollup) en proyectos web. Vite aprovecha los módulos ES nativos (`ESM`) implementados en los navegadores modernos para ofrecer un servidor de desarrollo casi instantáneo, sin necesidad de reempaquetar todo el código cada vez que se guarda un cambio.

---

## 4. ¿Por qué se seleccionó la plantilla Vanilla con TypeScript?
Se seleccionó la plantilla **Vanilla** para mantener la aplicación ligera, directa y enfocada en los fundamentos de la Web sin la abstracción ni la sobrecarga de un framework (como React, Angular o Vue). Se complementa con **TypeScript** para añadir tipado estático, autocompletado avanzado en la IDE, autodocumentación y la detección temprana de errores en tiempo de compilación.

---

## 5. ¿Cuál es la diferencia entre `pnpm install`, `pnpm dev` y `pnpm build`?
* **`pnpm install`**: Lee las dependencias declaradas en el archivo `package.json`, las descarga en la carpeta `node_modules` y crea o actualiza el archivo de bloqueo `pnpm-lock.yaml`.
* **`pnpm dev`**: Inicia el servidor local de desarrollo con recarga en tiempo real (*live-reloading*) para visualizar y trabajar en la aplicación mientras se edita el código.
* **`pnpm build`**: Ejecuta el compilador de TypeScript (`tsc`) y el empaquetador de Vite para transformar, minificar y optimizar todo el código fuente en archivos estáticos listos para producción.

---

## 6. ¿Qué información contiene `package.json`?
El archivo `package.json` es el manifiesto principal del proyecto. Contiene:
* **Metadatos del proyecto**: Nombre, versión, descripción, autor y licencias.
* **Scripts de automatización**: Comandos ejecutables como `dev`, `build`, `preview`.
* **Dependencias de producción (`dependencies`)**: Librerías necesarias para que la app funcione en ejecución.
* **Dependencias de desarrollo (`devDependencies`)**: Herramientas requeridas únicamente durante el desarrollo (compiladores, linters, types, etc.).

---

## 7. ¿Por qué debe conservarse `pnpm-lock.yaml` en el repositorio?
Debe conservarse porque registra y congela (*lock*) la versión exacta de cada paquete y subdependencia que se instaló en el proyecto. Subirlo al repositorio garantiza la **reproducibilidad del entorno**: asegura que todos los miembros del equipo y los servidores de despliegue utilicen exactamente las mismas versiones de las librerías, evitando fallos por inconsistencias entre entornos.

---

## 8. ¿Por qué `node_modules` no debe subirse a GitHub?
Porque es un directorio sumamente pesado que contiene miles de archivos y subcarpetas locales redundantes. Subirlo a GitHub saturaría inútilmente el repositorio de código. Dado que `package.json` y `pnpm-lock.yaml` ya especifican con exactitud qué paquetes se necesitan, cualquier desarrollador puede regenerar `node_modules` localmente ejecutando `pnpm install`.

---

## 9. ¿Cuál es la función de `main.ts`?
`main.ts` es el punto de entrada principal (*entrypoint*) de la lógica de TypeScript. Su función es inicializar la aplicación, enlazar la lógica de programación con la estructura del DOM en `index.html`, importar los estilos globales y bootstrapear/arrancar los componentes e integraciones principales.

---

## 10. ¿Qué ventaja ofrece separar el código en `components`, `models`, `services`, `styles` y `utils`?
Ofrece una arquitectura limpia basada en el **Principio de Separación de Intereses** (*Separation of Concerns*). Las principales ventajas son:
* **`components/`**: Mantiene encapsulada la interfaz de usuario y sus elementos interactivos.
* **`models/`**: Define las interfaces y tipos de datos de TypeScript.
* **`services/`**: Concentra la lógica de llamadas a APIs externas o bases de datos.
* **`styles/`**: Organiza los estilos CSS/SCSS de forma modular.
* **`utils/`**: Almacena funciones auxiliares puras y reutilizables.

Esta modularidad mejora la legibilidad, facilita la reutilización de código, reduce los conflictos al trabajar en equipo (*merge conflicts*) y simplifica las pruebas unitarias.

---

## 11. ¿Qué diferencia existe entre el código fuente almacenado en `src` y los archivos generados en `dist`?
* **`src` (Source)**: Contiene el código fuente legible, estructurado y modular escrito por los desarrolladores (archivos `.ts`, `.css`, etc.). No es apto para ejecutarse directamente en un navegador sin antes ser procesado.
* **`dist` (Distribution)**: Contiene el resultado compilado y optimizado que genera `pnpm build`. Son archivos HTML, CSS y JavaScript nativo (`.js`) unificados, minificados y optimizados para que cualquier navegador web los interprete a máxima velocidad.

---

## 12. ¿Qué error o dificultad encontraste durante la configuración y cómo lo resolviste?
* **Dificultad**: Al intentar ejecutar scripts de PNPM o Vite, el sistema operativo o la terminal mostraba errores de restricción de políticas de ejecución de PowerShell / falta de variables de entorno PATH para PNPM.
* **Solución**: Se habilitaron los permisos necesarios en la consola y se verificó que la versión de Node.js estuviera actualizada. Posteriormente, se reinstaló PNPM mediante el gestor Corepack de Node.js (`corepack enable pnpm`) para asegurar que el entorno estuviera correctamente vinculado.

---

## 13. ¿Cómo comprobaste que el repositorio puede ejecutarse en otro equipo?
Se realizó una prueba de entorno limpio:
1. Se clonó el repositorio remoto desde GitHub en un directorio independiente.
2. Se verificó que la carpeta `node_modules` no estuviera presente.
3. Se ejecutó `pnpm install` para restaurar todas las dependencias basándose únicamente en `pnpm-lock.yaml`.
4. Se inició el proyecto mediante `pnpm dev` comprobando que la aplicación abrió correctamente en la dirección local `http://localhost:5173` sin arrojar errores en la consola.

---

## 14. ¿Qué aprendizaje de esta actividad será necesario para continuar desarrollando GIFinder?
Comprender la importancia de la estructura modular de archivos en TypeScript, el uso estricto de interfaces (`models`) para mapear la respuesta JSON de la API de Giphy, el desarrollo de un módulo de comunicación (`services/giphy.service.ts`) para realizar peticiones `fetch`, y el manejo profesional del flujo de control de versiones con Git (respetando `.gitignore`).