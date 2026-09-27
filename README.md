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

## License

[CC0 1.0](LICENSE): use, copy and adapt the templates for any purpose, including client work, without attribution.
CC0 does not grant trademark rights.

Google Analytics is a trademark of Google LLC. Meta is a trademark of Meta Platforms, Inc. These templates and
Tracklint are not affiliated with, endorsed or sponsored by Google or Meta.

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
- **Licencia:** [CC0 1.0](LICENSE): úsalas, cópialas y adáptalas para lo que quieras, también en trabajos para
  clientes, sin atribución. CC0 no concede derechos sobre marcas.

Google Analytics es una marca de Google LLC. Meta es una marca de Meta Platforms, Inc. Estas plantillas y Tracklint no
están afiliados a Google ni a Meta, ni cuentan con su respaldo o patrocinio.
