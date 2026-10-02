# Preguntas de cierre - EC1 F3 A4

## 1. ¿Qué diferencia existe entre una operación síncrona y una asíncrona?

En una operación síncrona cada instrucción se ejecuta y termina antes de pasar a la siguiente, bloqueando el hilo principal mientras tanto. En una operación asíncrona, el programa inicia una tarea que puede tardar (como una petición de red) y sigue atendiendo otros eventos mientras espera el resultado. En GIFinder, transformar un arreglo con `.map()` dentro de `mapGiphyGif` es síncrono, mientras que `fetch(url)` dentro de `requestGifs` es asíncrono: la respuesta de GIPHY puede tardar e incluso fallar, y la aplicación sigue funcionando (mostrando el estado `Loading`) mientras la espera.

## 2. ¿Cuáles son los estados de una promesa y qué relación tienen con async/await?

Una promesa puede estar `pending` (en proceso, sin resultado todavía), `fulfilled` (se resolvió con un valor) o `rejected` (terminó en error). `async/await` es una forma de consumir esas promesas con una sintaxis parecida al código secuencial: `await` pausa la ejecución de la función hasta que la promesa deja de estar `pending`. Por ejemplo, en `requestGifs`, `await fetch(url)` espera a que la promesa de `fetch` pase a `fulfilled` (con la `Response`) o lance un error que se captura en el `try/catch` de `main.ts`.

## 3. ¿Qué devuelve fetch y qué devuelve response.json()?

`fetch(url)` devuelve una `Promise<Response>`: un objeto que representa la respuesta HTTP (con `status`, `ok`, cabeceras, etc.), pero todavía no contiene los datos ya interpretados. `response.json()` devuelve otra promesa, que al resolverse entrega el cuerpo de la respuesta ya convertido de JSON a un valor de JavaScript. Por eso en `requestGifs` se hace `await fetch(url)` primero y, una vez comprobado `response.ok`, se hace `await response.json()` para obtener el objeto `GiphyResponse`.

## 4. ¿Por qué es necesario comprobar response.ok?

Porque una respuesta HTTP con código de error (404, 401, 500, etc.) **no** hace que la promesa de `fetch` se rechace automáticamente; `fetch` solo falla por errores de red. Si no se revisa `response.ok`, el código intentaría leer `response.json()` sobre una respuesta de error y seguiría como si todo estuviera bien. Por eso `requestGifs` lanza explícitamente un `Error` con el `response.status` cuando `response.ok` es `false`, antes de intentar interpretar el cuerpo.

## 5. ¿Cómo se utilizan try, catch y unknown para manejar errores en GIFinder?

Tanto `loadTrending` como el listener `submit` de `main.ts` envuelven sus llamadas asíncronas (`getTrendingGifs` / `searchGifs`) en un bloque `try`. Si algo falla (red caída, clave inválida, estado distinto de 200 en `meta.status`), el `catch (error: unknown)` recibe el error. Como TypeScript tipa el error atrapado como `unknown` (no como `any`), `showRequestError` primero comprueba `error instanceof Error` antes de leer `error.message`, evitando asumir una forma del error que podría no cumplirse.

## 6. ¿Qué diferencia existe entre GiphyGif y Gif, y qué responsabilidad tiene mapGiphyGif?

`GiphyGif` (en `giphy-response.interface.ts`) describe la forma **externa**: los campos tal como los entrega la API de GIPHY, incluyendo cosas que GIFinder no necesita conservar, como las distintas variantes de imagen dentro de `images`. `Gif` (en `gif.interface.ts`) es el modelo **interno** y estable que usan los componentes (`gallery.ts`, `gif-detail.ts`): simple, con solo los campos que la interfaz realmente utiliza. `mapGiphyGif`, dentro de `gif.service.ts`, es la función que traduce de uno a otro: toma un `GiphyGif`, elige `fixed_width` u `original` según esté disponible, y arma un objeto `Gif` limpio con `url`, `detailUrl`, `altText`, etc.

## 7. ¿Por qué se utiliza URLSearchParams al construir la solicitud?

`URLSearchParams` arma el query string codificando automáticamente los valores (espacios, acentos, símbolos) con el formato que espera una URL, evitando errores de codificación manual. En `buildUrl`, se usa para combinar siempre `api_key`, `limit` y `rating` con los parámetros específicos de cada endpoint (por ejemplo `q` y `lang` en una búsqueda), generando una URL válida sin concatenar strings a mano.

## 8. ¿Qué significa Promise<Gif[]> en el tipo de retorno?

Significa que la función no devuelve un arreglo de `Gif` de inmediato, sino una promesa que **eventualmente** se resolverá con ese arreglo (o se rechazará con un error). `getTrendingGifs` y `searchGifs` están tipadas así porque dependen de una petición de red: quien las llama necesita usar `await` (o `.then`) para obtener el arreglo real de `Gif`, tal como se hace en `loadTrending` y en el listener `submit`.

## 9. ¿Qué diferencia existe entre .env.local y .env.example, y por qué una variable VITE_ no debe considerarse secreta?

`.env.local` contiene la clave real de GIPHY y está en `.gitignore`, por lo que nunca se sube al repositorio. `.env.example` solo documenta el nombre de la variable (`VITE_GIPHY_API_KEY=`) sin ningún valor, y sí se versiona, para que cualquier persona sepa qué variable debe configurar. Una variable con prefijo `VITE_` queda embebida en el código JavaScript que Vite entrega al navegador durante la compilación, así que cualquier persona que inspeccione las peticiones de red o el bundle final puede verla; por eso no debe usarse este patrón para contraseñas o tokens que requieran confidencialidad real, solo para una clave limitada de uso académico.

## 10. ¿Cómo comprobaste que .env.local no está versionado?

Ejecutando `git check-ignore -v .env.local`, que debe mostrar la regla de `.gitignore` que lo excluye (confirmando que Git lo está ignorando), y `git ls-files .env.local`, que no debe devolver ninguna salida (confirmando que el archivo no está siendo rastreado). Si este segundo comando llegara a mostrar el archivo, significaría que ya se había agregado antes de ignorarlo y habría que retirarlo del seguimiento antes de hacer `push`.

## 11. ¿Por qué Loading puede observarse con mayor claridad al consultar una API?

Porque ahora el estado `Loading` depende del tiempo real que tarda una solicitud de red en resolverse (latencia, velocidad de conexión, tamaño de la respuesta), que casi siempre toma al menos unos cientos de milisegundos. En la versión local (EC1 F2 A3), `searchGifs` filtraba un arreglo en memoria de forma instantánea, así que el estado `Loading` se activaba y desactivaba demasiado rápido para notarse. Con `fetch` a GIPHY, el `renderStatus(RequestStatus.Loading, status)` que se llama antes del `await` sí permanece visible el tiempo suficiente para que el usuario lo perciba.

## 12. ¿Qué dificultad se presentó durante la integración y cómo comprobaste que quedó resuelta?

*(Personaliza esta respuesta con tu experiencia real; aquí tienes una base que puedes ajustar):*

Una dificultad común en esta integración es que `fixed_width` puede no venir presente en la respuesta de GIPHY para algún GIF, lo que obligaría a usar `images.original` como respaldo (de ahí el operador `??` en `mapGiphyGif`). Otra dificultad frecuente es olvidar reiniciar `pnpm dev` después de crear `.env.local`, lo que hace que `import.meta.env.VITE_GIPHY_API_KEY` aparezca como `undefined` y se lance el error de `getApiKey`. Esto se comprobó como resuelto verificando en la pestaña Network del navegador que la solicitud a `api.giphy.com` respondía con estado 200 tanto para tendencias como para una búsqueda, y ejecutando `pnpm build` sin errores de TypeScript.
