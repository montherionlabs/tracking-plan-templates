# GA4 + Meta Pixel tracking plan templates (CSV)

Ready-to-edit measurement plans for **Google Analytics™ 4** and the **Meta Pixel**, as plain CSV files you can open in
any spreadsheet. They describe which events a site should send, which parameters each event needs, a few value rules,
how many times each event may fire and which consent it requires.

They are the same templates that ship inside [Tracklint](https://tracklint.netlify.app/), a Chrome DevTools panel for
tracking QA, so you can load them there to check a site's GA4 and Meta Pixel requests against the plan. You can also
use them on their own, as a checklist or as a starting point for a client's tracking plan.

*[Versión en español más abajo.](#plantillas-de-plan-de-medición-ga4--píxel-de-meta-csv)*

## Templates

| File | What it covers |
|---|---|
| [`ecommerce-ga4-meta.csv`](templates/ecommerce-ga4-meta.csv) | E-commerce funnel: `view_item`, `add_to_cart`, `begin_checkout`, `purchase` (GA4) and `ViewContent`, `AddToCart`, `InitiateCheckout`, `Purchase` (Meta) |
| [`ecommerce-eu-consent-ga4-meta.csv`](templates/ecommerce-eu-consent-ga4-meta.csv) | The same funnel for a site with a consent banner: GA4 events need `analytics_storage`, Meta events need `ad_storage`, plus `PageView` |
| [`lead-generation-ga4-meta.csv`](templates/lead-generation-ga4-meta.csv) | Lead forms: `generate_lead` (GA4) and `Lead` (Meta) |

Each one also comes in Spanish (`.es.csv`), with the notes translated and `;` as the separator, which Excel recognizes
directly when its regional settings use a semicolon as the list separator. Required parameters follow Google's GA4
e-commerce reference and Meta's standard events reference; adapt them to your site.

## Columns

| Column | Meaning |
|---|---|
| `platform` | `ga4` or `meta` |
| `event` | Event name exactly as sent (`purchase`, `Purchase`...) |
| `required` | Parameters that must be present, separated by `;` (for example `transaction_id;currency;value;items`) |
| `rules` | Value rules separated by `;` (see below) |
| `count` | `once`, `at_least_once`, `any` or `never` (an event that must never fire) |
| `consent` | Consent types the event needs, such as `analytics_storage` or `ad_storage` |
| `note` | Free text for people (for example, the page where the event should fire) |

Only `platform` and `event` are required columns; all other columns are optional, including `id` (the GA4
measurement ID or Meta pixel ID the event must go to) and `url` (text the page URL must contain, or a pattern with
`*`). An empty or omitted `count` defaults to `at_least_once`. Accepted consent types: `ad_storage`,
`analytics_storage`, `ad_user_data`, `ad_personalization`; separate multiple types with `;`.

A line starting with `#` is a comment. `# name: My plan` sets the plan name, and `# consent_mode: advanced` (or
`basic`) tells Tracklint which Consent Mode the site uses; in advanced mode, GA4 cookieless pings with storage denied
are informational, not errors.

## Rules

- `currency is iso4217`, `value is number`, `transaction_id present`, `coupon absent`
- `value > 0`, `value >= 0`, `quantity <= 10`, `currency = "EUR"`, `payment_type != "test"`
- `currency in EUR|USD|MXN`, `item_brand not in test|demo`
- `page_location starts with https://`, `page_location ends with /thanks`, `page_location contains checkout`,
  `page_location like https://*/checkout*`
- `each item has item_id|item_name` (every product in `items` has at least one of those fields)

## Using them with Tracklint

Open Tracklint's panel in Chrome DevTools, load the CSV in the validation tab, record a journey through the site and
review the findings. The free version shows a preview of the validation (the summary, each plan row's status and one
first finding); Pro shows every finding. Tracklint checks the requests the browser sends during the journey you record;
it does not verify processing inside Google Analytics or Meta, server-to-server integrations or legal compliance.

## Checking a plan or decoding a request with the Tracklint MCP server (optional)

If you use an AI client that supports the Model Context Protocol (Claude Desktop, Claude Code, Cursor or another stdio
client), the free [`@montherion/tracklint-mcp`](https://www.npmjs.com/package/@montherion/tracklint-mcp) server (`0.2.0`)
provides two local read-only tools:

- **`tracklint_decode_request`:** takes `url` (required), `body` and `method` (`GET` or `POST`) and decodes supported
  GA4 (`/g/collect`, including server-side GTM), Meta Pixel (`facebook.com/tr`) and TikTok Pixel (`/api/v2/pixel`)
  requests without sending them. It returns each event's platform, standard event and parameter names, amounts, ISO 4217
  currencies, item counts, `user_data` SHA-256 format checks, GA4 request consent signals (`gcs`/`gcd`,
  `source: "request_signals"`) and warning codes (`PURCHASE_WITHOUT_TRANSACTION_ID`, `PURCHASE_WITHOUT_EVENT_ID`,
  `VALUE_WITHOUT_CURRENCY`, `CURRENCY_NOT_ISO4217`, `USER_DATA_NOT_HASHED`, `EMAIL_IN_VALUE`, `PHONE_IN_VALUE`,
  `PHONE_IN_LINK`, `WHATSAPP_LINK`, `SECRET_LIKE_VALUE`, `USER_IDENTIFIERS_PRESENT`).
- **`tracklint_check_plan`:** checks these CSV files (comma, semicolon or tab) and returns each row's structure
  (platform, event, ID presence and format, required parameters, rule types, counts, URL filter type, consent types) and
  every error with its code and line number. It checks the plan itself, not a recorded journey. Version `0.2.0` accepts
  `google_ads` rows: to match conversions from the reader, use `conversion` as the event, and in `id` select either one
  action (`AW-ID/LABEL`) or all actions observed under the same `AW-ID` (other non-empty event names are accepted as
  custom events; `valid` means the plan reader found no errors, so also check `id.formatValid`). The directive
  `# google_ads_purchase: AW-ID/LABEL` is reported only as a count (`googleAdsPurchaseActions`; invalid format returns
  `bad_ads_purchase`). Google Ads requests are not decoded in `0.2.0`.

Custom names and free-text values from the request or the plan are never returned as text, only as position, type and
length or as a warning code. Measurement, pixel and Google Ads IDs are reported only through `present` and
`formatValid`. Every limit returns an explicit error code (`REQUEST_TOO_LARGE`, `TOO_MANY_EVENTS`,
`TOO_MANY_PARAMETERS`, `PLAN_TOO_MANY_ROWS`, `PLAN_COMMENT_REJECTED`, `PLAN_DUPLICATE_HEADER`, `INCOMPLETE_REQUEST`,
`PROCESSING_TIMEOUT`, `RESPONSE_TOO_LARGE`); nothing is truncated silently. This version does not include HAR file
audits, journey validation, exports or Freemius integration.

Requires Node.js 22.23.4 or later within the 22.x line, or 26.9.0 or later within the 26.x line (tested on Node.js
22.23.4 and 26.9.0; no environment variables or keys). Example for Claude Desktop
([how to connect a local MCP server](https://modelcontextprotocol.io/docs/develop/connect-local-servers)); other
clients may need the same settings in their own format:

```json
{"mcpServers": {"tracklint": {"command": "npx", "args": ["-y", "@montherion/tracklint-mcp@0.2.0"]}}}
```

Then ask your client, for example: *Check this tracking plan with tracklint_check_plan*, and paste the CSV, or *Decode
this request with tracklint_decode_request*, and paste the URL and body.

Tracklint MCP 0.2.0 processes URLs, request bodies and CSV text received from the client in memory. Its tools do not
read user files, make network requests or save inputs or results to disk. Tracklint does not receive this content on
its own servers. The client and its AI provider may receive and retain tool arguments and results under their policies.
Share only data you are authorized to share.

## License

[CC0 1.0](LICENSE): use, copy and adapt the templates for any purpose, including client work, without attribution.
CC0 does not grant trademark rights.

Google Analytics and Google Ads are trademarks of Google LLC. Meta is a trademark of Meta Platforms, Inc. TikTok and
Claude are trademarks of their respective owners. These templates and Tracklint are not affiliated with, endorsed or
sponsored by Google, Meta, TikTok or Anthropic.

---

# Plantillas de plan de medición GA4 + píxel de Meta (CSV)

Planes de medición listos para editar para **Google Analytics™ 4** y el **píxel de Meta**, en archivos CSV que se abren
con cualquier hoja de cálculo. Indican qué eventos debe enviar una web, qué parámetros necesita cada uno, algunas
reglas de valores, cuántas veces puede enviarse cada evento y qué consentimiento requiere.

Son las mismas plantillas que incluye [Tracklint](https://tracklint.netlify.app/es.html), un panel de las DevTools de
Chrome para hacer QA de tracking: puedes cargarlas allí para comprobar contra el plan las peticiones de GA4 y del
píxel de Meta de una web, o usarlas por separado como lista de comprobación o punto de partida del plan de un cliente.

- **Archivos:** los de la tabla de arriba; los que terminan en `.es.csv` están en español y usan `;` como separador
  (Excel lo reconoce directamente cuando su configuración regional usa el punto y coma como separador de listas).
- **Columnas:** `platform` (`ga4` o `meta`), `event`, `required` (parámetros obligatorios separados por `;`), `rules`
  (reglas separadas por `;`), `count` (`once`, `at_least_once`, `any` o `never`), `consent` (`analytics_storage`,
  `ad_storage`...) y `note` (nota libre). Solo `platform` y `event` son columnas obligatorias; las demás son opcionales, también `id` y `url`.
  Si `count` está vacío o se omite, se aplica `at_least_once`. Consentimientos admitidos: `ad_storage`,
  `analytics_storage`, `ad_user_data`, `ad_personalization`; separa varios con `;`. Las líneas que empiezan por `#`
  son comentarios; `# name: Mi plan` establece el nombre del plan y `# consent_mode: advanced` (o `basic`) indica a
  Tracklint el modo de consentimiento de la web; en modo avanzado, los pings GA4 sin cookies con almacenamiento
  denegado son informativos, no errores.
- **Reglas:** las de la lista de arriba (`value > 0`, `currency is iso4217`, `currency in EUR|USD|MXN`,
  `each item has item_id|item_name`...). Estas plantillas mantienen los identificadores en inglés y traducen las
  notas; el parser también admite alias en español.
- **Con Tracklint:** abre su panel en las DevTools, carga el CSV en la pestaña de validación, graba un recorrido por la
  web y revisa los hallazgos. La versión gratis muestra una vista previa de la validación (el resumen, el estado de
  cada fila del plan y un primer hallazgo); Pro muestra todos los hallazgos. Tracklint revisa las peticiones que envía
  el navegador durante el recorrido que grabas, no el procesamiento dentro de Google Analytics o Meta, las
  integraciones de servidor a servidor ni el cumplimiento legal.

## Comprobar un plan o decodificar una petición con el servidor MCP de Tracklint (opcional)

Si usas un cliente de IA compatible con el Model Context Protocol (Claude Desktop, Claude Code, Cursor u otro cliente
stdio), el servidor gratuito [`@montherion/tracklint-mcp`](https://www.npmjs.com/package/@montherion/tracklint-mcp)
(`0.2.0`) ofrece dos herramientas locales de solo lectura:

- **`tracklint_decode_request`:** recibe `url` (obligatoria), `body` y `method` (`GET` o `POST`) y decodifica sin
  reenviarlas peticiones compatibles de GA4 (`/g/collect`, también desde GTM del lado del servidor), del píxel de Meta
  (`facebook.com/tr`) y del píxel de TikTok (`/api/v2/pixel`). Devuelve por cada evento la plataforma, nombres de
  eventos y parámetros estándar, importes, monedas ISO 4217, número de productos, comprobación de formato SHA-256 en
  `user_data`, señales de consentimiento que transporta la petición de GA4 (`gcs`/`gcd`, `source: "request_signals"`) y
  códigos de aviso (`PURCHASE_WITHOUT_TRANSACTION_ID`, `PURCHASE_WITHOUT_EVENT_ID`, `VALUE_WITHOUT_CURRENCY`,
  `CURRENCY_NOT_ISO4217`, `USER_DATA_NOT_HASHED`, `EMAIL_IN_VALUE`, `PHONE_IN_VALUE`, `PHONE_IN_LINK`,
  `WHATSAPP_LINK`, `SECRET_LIKE_VALUE`, `USER_IDENTIFIERS_PRESENT`).
- **`tracklint_check_plan`:** comprueba estos CSV (coma, punto y coma o tabulador) y devuelve la estructura de cada fila
  (plataforma, evento, presencia y formato del ID, parámetros obligatorios, tipos de regla, repeticiones, tipo de filtro
  de URL y tipos de consentimiento) y cada error con su código y número de línea. Comprueba el plan, no un recorrido
  grabado. Desde la `0.2.0` acepta filas `google_ads`: para que coincidan con las conversiones del lector, usa
  `conversion` como evento, y en `id` selecciona una acción (`AW-ID/ETIQUETA`) o todas las acciones observadas bajo un
  mismo `AW-ID` (otros nombres no vacíos se aceptan como personalizados; `valid` indica que el lector de planes no
  encontró errores, por lo que conviene consultar también `id.formatValid`). La directiva
  `# google_ads_purchase: AW-ID/ETIQUETA` se informa solo como recuento (`googleAdsPurchaseActions`; un formato
  incorrecto da `bad_ads_purchase`). Esta versión no decodifica peticiones de Google Ads.

Ningún texto escrito por la página inspeccionada ni por el autor del plan se devuelve como texto, solo por posición,
tipo y longitud o como código de aviso. Los IDs de medición, de píxel y de Google Ads se informan únicamente mediante
`present` y `formatValid`. Cada límite devuelve un código de error explícito (`REQUEST_TOO_LARGE`, `TOO_MANY_EVENTS`,
`TOO_MANY_PARAMETERS`, `PLAN_TOO_MANY_ROWS`, `PLAN_COMMENT_REJECTED`, `PLAN_DUPLICATE_HEADER`, `INCOMPLETE_REQUEST`,
`PROCESSING_TIMEOUT`, `RESPONSE_TOO_LARGE`); ningún resultado se recorta en silencio. Esta versión no incluye auditoría
de archivos HAR, validación de recorridos, exportaciones ni integración con Freemius.

Requiere Node.js 22.23.4 o posterior de la rama 22, o 26.9.0 o posterior de la rama 26 (probado en Node.js 22.23.4 y
26.9.0; sin variables de entorno ni claves). Ejemplo para Claude Desktop
([cómo conectar un servidor MCP local](https://modelcontextprotocol.io/docs/develop/connect-local-servers)); otros
clientes pueden necesitar los mismos datos en su propio formato:

```json
{"mcpServers": {"tracklint": {"command": "npx", "args": ["-y", "@montherion/tracklint-mcp@0.2.0"]}}}
```

Después pide a tu cliente, por ejemplo: *Comprueba este plan de medición con tracklint_check_plan*, y pega el CSV, o
*Decodifica esta petición con tracklint_decode_request*, y pega la URL y el cuerpo.

Tracklint MCP 0.2.0 procesa en memoria las URL, cuerpos de peticiones y CSV que recibe del cliente. Sus herramientas no
leen archivos del usuario, no realizan peticiones de red ni guardan entradas o resultados en disco. Tracklint no recibe
esos contenidos en servidores propios. El cliente y su proveedor de IA pueden recibir y conservar argumentos y
resultados según sus políticas. Comparte únicamente datos que estés autorizado a facilitar.

- **Licencia:** [CC0 1.0](LICENSE): úsalas, cópialas y adáptalas para lo que quieras, también en trabajos para
  clientes, sin atribución. CC0 no concede derechos sobre marcas.

Google Analytics y Google Ads son marcas de Google LLC. Meta es una marca de Meta Platforms, Inc. TikTok y Claude son
marcas de sus respectivos propietarios. Estas plantillas y Tracklint no están afiliados a Google, Meta, TikTok ni
Anthropic, ni cuentan con su respaldo o patrocinio.
