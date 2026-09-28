# Azure Price Scraper — ejercicio con HTTP Trigger

Proyecto académico de Python y Azure Functions para consultar el nombre y el precio de un producto en una página con una estructura HTML concreta. Devuelve una respuesta de texto con los campos extraídos, la fecha de consulta y la URL.

Desarrollado por Carlos Falconi durante clases guiadas del Máster de ESESA y posteriormente incorporado a su portafolio. El autor realizó el ejercicio de clase sin generación de código por IA; la documentación se preparó y revisó posteriormente con asistencia de IA.

## Qué implementa

- Una Azure Function con la ruta `http_trigger_precios_web`.
- Lectura de una URL desde el parámetro `url`; si falta, utiliza la página de ejemplo de AFT Grupo definida en el código.
- Descarga del HTML mediante `requests.get`, con timeout de 10 segundos y comprobación de errores HTTP.
- Extracción mediante `str.find`, índices y recortes de texto. **No utiliza BeautifulSoup ni un navegador automatizado.**
- Respuesta con producto, precio, timestamp y URL. El precio se conserva como texto, no se convierte a un valor numérico.
- Respuesta HTTP 502 cuando la descarga produce una excepción de `requests`.

## Flujo

Petición HTTP con URL → Azure Function → descarga del HTML → búsqueda de marcadores → respuesta de texto.

El extractor depende de estos marcadores literales:

| Campo | Inicio | Final |
|---|---|---|
| Nombre | `<strong>DESCRIPCIÓN:</strong><br>` | `<br />` |
| Precio | `<div class="precio_ficha">` | `</div>` |

**Cambiar la URL no convierte el extractor en compatible con cualquier web.** Si cambia el HTML, puede ser necesario adaptar el código. No ejecuta JavaScript para cargar contenido dinámico.

## Entrada y salida

Ruta de ejemplo para una Function App configurada:

```text
GET https://<function-app-host>/api/http_trigger_precios_web?url=<product-url>
```

Formato ilustrativo de respuesta; no es un precio real ni una prueba de disponibilidad actual:

```text
Producto:   Example product
Precio:     12,50 €
Timestamp:  2026-04-15 10:32:01
URL:        https://example.com/product
```

Si no encuentra los marcadores, devuelve `No encontrado` para el campo correspondiente, pero mantiene HTTP 200. Por tanto, un estado 200 no garantiza que haya extraído ambos campos. El timestamp utiliza la hora del entorno de ejecución, sin zona horaria explícita.

## Archivos principales

| Archivo | Contenido |
|---|---|
| `function_app.py` | Función HTTP y lógica de extracción |
| `2026_04_15_obtener_precio_de_web_para_azure_functions.ipynb` | Notebook del ejercicio |
| `host.json` | Configuración del runtime |
| `requirements.txt` | Dependencias declaradas; véase la nota siguiente |
| `.github/workflows/main_httptriggerpriceweb.yml` | Workflow de construcción y despliegue a Azure |
| `Captura de pantalla.png`, `Captura de pantalla 2.png` | Capturas del proyecto; no acreditan disponibilidad actual |

## Ejecución local y estado del despliegue

El workflow del repositorio especifica Python 3.11. Para probar localmente se necesita un entorno Python compatible y Azure Functions Core Tools instalado y configurado.

`requirements.txt` incluye `logging` y `datetime`, aunque ambos pertenecen a la biblioteca estándar de Python. Ese archivo requiere limpieza antes de reutilizar el flujo de instalación/despliegue. Para una prueba local aislada, las dependencias externas utilizadas por la función son `azure-functions` y `requests`:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install azure-functions requests
func start
```

Con el servicio local iniciado, se puede consultar la URL de ejemplo ya definida en el código:

```bash
curl "http://localhost:7071/api/http_trigger_precios_web"
```

Esta llamada descarga una página externa. Verificar las condiciones del sitio antes de usarlo y evitar peticiones repetitivas. La extracción actual de esa web y el arranque del runtime no se han revalidado en la revisión documental del 28/09/2026.

**Atención al workflow:** un push a `main` o una ejecución manual pueden iniciar el despliegue configurado a Azure. No contiene una etapa de tests automatizados. Revisar recursos, credenciales y costes antes de ejecutarlo; no equivale a una garantía de que el endpoint esté activo hoy.

## Limitaciones antes de exponerlo públicamente

- La función declara acceso anónimo y acepta una URL suministrada por el usuario sin limitar los destinos. Antes de exponerla, restringir destinos y redirecciones a los necesarios, bloquear acceso a redes internas y revisar autenticación y límites de peticiones.
- No existe validación robusta del HTML ni normalización de precios.
- Los errores de descarga se incluyen en la respuesta; revisar qué detalles se muestran al cliente.
- No se han implementado almacenamiento histórico, alertas, ejecución periódica ni seguimiento automático de cambios de precio.
- Es un prototipo académico, no una plataforma de monitorización de precios en producción.

## Mejoras pendientes

- [ ] Limpiar y fijar dependencias después de probarlas.
- [ ] Añadir tests con HTML de ejemplo, campos ausentes y errores de descarga.
- [ ] Restringir las URL aceptadas y revisar el acceso antes de un despliegue público.
- [ ] Mejorar el parser y la validación de resultados.
- [ ] Incorporar histórico, programación periódica y alertas si el caso de uso lo requiere.

## Qué permite explicar este proyecto

HTTP trigger, parámetros de consulta, petición a un servicio externo, manejo de excepciones y extracción sencilla mediante operaciones de strings. Su alcance es deliberadamente pequeño: consultar una página compatible y devolver los datos extraídos.
