# Preguntas de cierre - EC1 F2 A3

## 1. ¿Qué significa refactorizar una aplicación?

Refactorizar significa reorganizar y mejorar la estructura interna del código sin cambiar su comportamiento externo. La aplicación debe seguir funcionando exactamente igual para el usuario, pero por dentro el código queda más limpio, ordenado, tipado y fácil de mantener. En este proyecto, refactorizar implicó pasar de JavaScript "plano" (o de un solo archivo) a TypeScript organizado en módulos, con tipos definidos, funciones puras separadas de la manipulación del DOM, y responsabilidades claras para cada archivo.

## 2. ¿Por qué el proyecto se dividió en módulos?

Se dividió en módulos para separar responsabilidades: cada archivo se encarga de una sola cosa (por ejemplo, un módulo para llamar a la API, otro para los tipos/interfaces, otro para las funciones que operan sobre los datos, y otro para manipular el DOM). Esto trae varias ventajas:
- El código es más fácil de leer y entender.
- Es más sencillo de probar y depurar, porque los errores quedan acotados a un módulo.
- Se reduce el acoplamiento: si cambio cómo pinto los resultados en pantalla, no toco la lógica de búsqueda.
- Permite reutilizar funciones en distintas partes de la app sin duplicar código.

## 3. ¿Cuál es la responsabilidad de main.ts?

`main.ts` actúa como el punto de entrada (orquestador) de la aplicación. No contiene la lógica de negocio en sí, sino que:
- Importa las funciones y tipos de los demás módulos.
- Conecta los eventos de la interfaz (por ejemplo, el formulario de búsqueda) con las funciones correspondientes.
- Coordina el flujo general: recibe la entrada del usuario, llama a las funciones que consultan/filtran los datos, y pasa el resultado a las funciones que actualizan el DOM.

En resumen, `main.ts` es el "director de orquesta" que conecta todas las piezas, pero delega el trabajo pesado a los módulos especializados.

## 4. ¿Qué diferencias existen entre una interfaz, un tipo unión y una enumeración?

- **Interfaz (`interface`)**: describe la forma/estructura de un objeto (qué propiedades tiene y de qué tipo son). Se usa para tipar objetos complejos, como un GIF con `id`, `title`, `url`, etc. Las interfaces se pueden extender y son ideales para modelar entidades.
- **Tipo unión (`type A | B`)**: permite que una variable acepte uno de varios tipos posibles. Por ejemplo, un estado que puede ser `"loading" | "success" | "error"`, o un valor que puede ser `string | undefined`. Es útil para representar variantes o estados alternativos.
- **Enumeración (`enum`)**: define un conjunto fijo y con nombre de valores constantes relacionados entre sí (por ejemplo, los distintos estados de la aplicación: `Loading`, `Success`, `Error`). A diferencia del tipo unión, un `enum` genera un objeto real en tiempo de ejecución y sus valores tienen nombres semánticos reutilizables en todo el código.

## 5. ¿Para qué se utiliza import type?

`import type` se usa para importar exclusivamente tipos, interfaces o enumeraciones usadas solo para el chequeo estático de TypeScript, dejando claro que esa importación no trae código ejecutable (no genera una importación real en el JavaScript compilado). Esto ayuda a:
- Evitar importaciones innecesarias en el bundle final (mejor rendimiento).
- Dejar explícito, para quien lea el código, qué se está importando solo con fines de tipado y qué se está importando como funcionalidad real.

## 6. ¿Dónde se aplicaron la desestructuración, spread y rest?

- **Desestructuración**: al extraer propiedades específicas de los objetos GIF devueltos por la API (por ejemplo, sacar `id`, `title` y `url` directamente de la respuesta) y al recibir parámetros de funciones que reciben un objeto de opciones.
- **Spread (`...`)**: al crear copias de arreglos o de objetos sin mutar el original (por ejemplo, al agregar un nuevo GIF a la colección existente: `[...gifs, nuevoGif]`), y al combinar/actualizar el estado de la aplicación.
- **Rest (`...`)**: al recibir un número variable de argumentos en alguna función utilitaria, o al desestructurar un objeto y agrupar las propiedades restantes en una sola variable (por ejemplo, separar una propiedad puntual y dejar "el resto" del objeto agrupado).

## 7. ¿Por qué searchGifs recibe la colección como parámetro?

Porque así la función se mantiene **pura**: en lugar de depender de una variable global o de ir a buscar los datos por su cuenta, recibe la colección de GIFs como argumento y devuelve un nuevo resultado filtrado, sin efectos secundarios. Esto tiene varias ventajas:
- Facilita las pruebas (se le puede pasar cualquier colección de prueba).
- Hace explícito de dónde vienen los datos que usa.
- Permite reutilizar la función con distintas fuentes de datos (por ejemplo, resultados ya cacheados) sin acoplarla a una sola fuente fija.

## 8. ¿Por qué findGifById puede devolver undefined?

Porque no hay garantía de que exista un GIF con el `id` buscado dentro de la colección: el usuario (o el propio código) podría pedir un identificador que no está presente, ya sea porque el elemento fue eliminado, la colección cambió, o simplemente se pasó un id incorrecto. TypeScript obliga a contemplar ese caso devolviendo `Gif | undefined` en vez de asumir que siempre habrá un resultado, evitando así errores en tiempo de ejecución al intentar usar un valor inexistente.

## 9. ¿Qué función cumple data-gif-id?

El atributo `data-gif-id` se usa para asociar cada elemento del DOM (por ejemplo, cada tarjeta de GIF renderizada) con el identificador único del GIF que representa. Esto permite, al capturar un evento de clic (u otro) sobre ese elemento, leer el `id` directamente desde el `dataset` del elemento y usarlo para buscar el GIF correspondiente en la colección (por ejemplo, con `findGifById`), sin necesidad de guardar referencias adicionales o de recorrer el DOM de otra forma.

## 10. ¿Qué es la delegación de eventos?

Es una técnica que consiste en asignar un único listener a un elemento contenedor (padre), en lugar de agregar un listener individual a cada uno de sus elementos hijos. Cuando ocurre un evento en un hijo (por ejemplo, un clic en una tarjeta de GIF), este "burbujea" hasta el contenedor, y ahí se identifica cuál fue el elemento específico que originó el evento (normalmente usando `event.target` y atributos como `data-gif-id`). Ventajas principales:
- Mejor rendimiento, sobre todo con listas dinámicas o muy grandes.
- Funciona automáticamente con elementos que se agregan después (no hay que volver a registrar listeners cada vez que se renderiza la lista).

## 11. ¿Por qué el estado Loading podría no observarse?

Porque si la respuesta de la API llega muy rápido, el cambio de estado a `Loading` y luego a `Success` (o `Error`) puede ocurrir en cuestión de milisegundos, demasiado rápido para que el ojo humano lo perciba en la interfaz. El renderizado del estado de carga puede incluso quedar "opacado" por el siguiente render si no se fuerza una actualización visual intermedia o un pequeño delay, dando la sensación de que el estado de carga nunca se mostró.

## 12. ¿Qué dificultad se presentó durante la refactorización y cómo se resolvió?

*(Esta respuesta conviene personalizarla con tu experiencia real. Aquí tienes una redacción base que puedes ajustar):*

Una de las principales dificultades al refactorizar fue **tipar correctamente los datos que llegan desde la API** (evitar el uso de `any` y definir interfaces que reflejaran la forma real de la respuesta), así como **separar la lógica pura de la manipulación del DOM** sin romper el funcionamiento original de la app. Esto se resolvió definiendo interfaces específicas para los datos de los GIFs y para los distintos estados de la aplicación, y organizando el código en módulos con responsabilidades bien delimitadas, probando cada función de forma aislada antes de integrarla nuevamente al flujo principal en `main.ts`.