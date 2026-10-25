# Consulta CTE

Web para que técnicos y arquitectos busquen y consulten los **Documentos Básicos oficiales del Código Técnico de la Edificación**. Todas las respuestas salen directamente del texto oficial, sin IA. Cada resultado indica la cita exacta (documento, sección, apartado, tabla y página), abre el PDF oficial en esa página con el pasaje resaltado y, cuando ayuda, muestra la respuesta de forma gráfica.

## Qué hace

- **Búsqueda en el texto oficial vigente** de la Parte I y de los DB SE, SE-AE, SE-C, SE-A, SE-F, SE-M, SI, SUA, HS, HR y HE, descargados de [codigotecnico.org](https://www.codigotecnico.org). Entiende términos coloquiales («barandilla» → «barrera de protección», «parking» → «aparcamiento»…) y referencias como «SUA 1», «apartado 4.2.1» o «tabla 3.1».
- **Cita exacta** de cada pasaje: `CTE DB SUA, Sección SUA 1, apartado 4.2.1 «Peldaños», págs. 11–12`, con botón para copiarla.
- **Visor del PDF oficial** que abre la página citada y resalta las líneas exactas del pasaje.
- **Respuestas gráficas**: esquemas acotados (escalera, rampa, barrera de protección, recorrido de evacuación), gráficos de barras, tablas, rangos de valores admisibles y comprobaciones paso a paso.
- **Fichas rápidas** para las consultas más habituales (escaleras, rampas, barandillas, recorridos de evacuación, transmitancias HE 1, sobrecargas de uso, resistencia al fuego de la estructura, aislamiento acústico, densidades de ocupación). Sus valores proceden del texto oficial y cada ficha enlaza sus citas literales, que se verifican automáticamente contra el corpus.
- **Explorador de documentos** con el índice de cada DB para abrir cualquier apartado.
- Diseño adaptable a móvil y modo claro/oscuro.

## Puesta en marcha

Requisitos: Node.js 20.12 o superior.

```bash
npm install
npm run preparar   # descarga los PDF oficiales (~20 MB) y genera data/corpus.json
npm start          # http://localhost:3000
```

## Publicar en Render

El repositorio incluye `render.yaml`, con el servicio ya configurado: plan gratuito, región de Frankfurt, construcción `npm ci && npm run preparar`, arranque `npm start`, `HOST=0.0.0.0` y `CTE_PROXY=1`.

1. Sube el proyecto a un repositorio de GitHub (puede ser privado).
2. En [render.com](https://render.com), entra con tu cuenta de GitHub y elige **New → Blueprint**. Selecciona el repositorio.
3. Render pedirá los datos del titular (`CTE_TITULAR`, `CTE_NIF`, `CTE_DOMICILIO`, `CTE_EMAIL`). Rellénalos y pulsa **Apply**.
4. La primera construcción tarda unos minutos: descarga los PDF oficiales y genera el índice. Después la web queda en `https://<nombre>.onrender.com`.
5. Cada `git push` a la rama `main` vuelve a desplegarla.

En el plan gratuito el servicio se suspende tras unos 15 minutos sin visitas y la primera petición posterior tarda alrededor de un minuto. Los planes de pago lo mantienen siempre activo. Para AdSense, añade tu dominio en **Settings → Custom Domains**; Render emite el certificado HTTPS automáticamente.

## Seguridad

- **Solo este equipo por defecto**: el servidor escucha en `127.0.0.1`. Para publicarlo en tu red o en internet, define `HOST=0.0.0.0`.
- **Solo lectura**: la API solo admite consultas (GET); no hay formularios, cuentas ni datos de usuarios que proteger.
- **Cabeceras**: política de contenido (CSP) estricta con *nonce* por petición: solo se ejecutan los scripts de la página y los que estos cargan (`'strict-dynamic'`), como recomienda Google para AdSense. Bloqueo de inserción en marcos ajenos (`frame-ancestors 'none'`, `X-Frame-Options`), `nosniff`, `COOP`, `Permissions-Policy`, y HSTS cuando la conexión es HTTPS.
- **Errores sin detalles internos**: las respuestas de error no muestran trazas ni rutas del disco.
- Búsquedas limitadas a `CTE_BUSQUEDAS_MINUTO` (120) por IP y minuto.

Si la publicas en internet, ponla detrás de un proxy inverso con HTTPS (Caddy, nginx…) y define `CTE_PROXY=1` para que el límite de búsquedas use la IP real del visitante. La configuración opcional está en `.env.example`.

## Publicidad (Google AdSense)

La página carga el código de AdSense de la cuenta `ca-pub-1133184139072632` (en `public/index.html`) y publica `/ads.txt` autorizando esa cuenta. Para que se muestren anuncios:

1. **Dominio aprobado**: AdSense solo sirve anuncios en el dominio que hayas añadido y Google haya revisado; en `localhost` no aparecen. Activa los *anuncios automáticos* en tu panel de AdSense o crea bloques de anuncio.
2. **Consentimiento (RGPD)**: para mostrar anuncios a visitantes del Espacio Económico Europeo y del Reino Unido, Google exige una plataforma de gestión del consentimiento certificada. La más sencilla es la propia de Google: AdSense → *Privacidad y mensajes* → *RGPD*. El aviso de cookies aparece entonces automáticamente, sin cambios en el código.
3. **Aviso legal, privacidad y cookies**: el aviso legal está en `/aviso-legal` (`public/aviso-legal.html`) y la política en `/privacidad` (`public/privacidad.html`), enlazada al pie de todas las páginas junto con el botón *Configuración de privacidad y cookies*, que reabre el aviso de Google. Rellena en `.env` los datos del titular (`CTE_TITULAR`, `CTE_NIF`, `CTE_DOMICILIO`, `CTE_EMAIL`, `CTE_ALOJAMIENTO` y, si es una sociedad, `CTE_REGISTRO`); mientras falten, las páginas los marcan como pendientes y el servidor lo avisa al arrancar. Es una plantilla: conviene que la revise un profesional y actualizarla si añades servicios (por ejemplo, analítica).

Si cambias de cuenta, actualiza el `client=` del script y el `pub-` de `public/ads.txt`.

## Actualizar los documentos

Cuando el Ministerio publique una nueva versión de algún DB:

```bash
npm run descargar -- --forzar
npm run indexar
npm test           # comprueba la búsqueda y que las citas de las fichas siguen existiendo
```

Las URL de los PDF están en `server/catalog.js`.

## Estructura

| Ruta | Contenido |
|---|---|
| `scripts/download.js` | Descarga de los PDF oficiales a `data/pdf/` |
| `scripts/ingest.js` | Extracción con pdf.js: secciones, apartados, tablas, figuras, página impresa y coordenadas de cada línea |
| `scripts/comprobar.js` | Comprobaciones de calidad (`npm test`) |
| `server/buscador.js` | Índice BM25 (MiniSearch) con lematización ligera, sinónimos y detección de referencias |
| `server/fichas.js` | Fichas rápidas con citas literales verificadas |
| `server/seguridad.js` | Cabeceras de seguridad, límite de búsquedas y gestión de errores |
| `public/js/visor.js` | Visor del PDF con resaltado del pasaje |
| `public/js/visuales.js` | Esquemas, gráficos y tablas |

## Limitaciones

- Las versiones consolidadas publicadas en codigotecnico.org **no tienen valor jurídico**; la referencia legal es la publicada en el BOE. La herramienta es de apoyo: verifica siempre en el documento oficial.
- Algunos símbolos de fuentes especiales del PDF (p. ej. «≤» en ciertas tablas del DB HR) no se extraen como texto; el visor muestra siempre el original.
- Los esquemas son orientativos y sin escala; las cotas reproducen los valores del texto citado.
- No se indexan los Documentos de Apoyo (DA) ni las versiones comentadas; pueden añadirse en `server/catalog.js`.
