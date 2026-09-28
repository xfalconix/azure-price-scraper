# 🌐 Azure Price Scraper — HTTP Trigger

<a href="https://azure.microsoft.com/services/functions"><img src="https://img.shields.io/badge/Azure%20Functions-0062AD?style=flat-square&logo=microsoftazure&logoColor=white" alt="Azure Functions"/></a>
<a href="https://pypi.org/project/requests/"><img src="https://img.shields.io/badge/requests-FF6849?style=flat-square&logo=python&logoColor=white" alt="requests"/></a>
<a href="https://github.com/xfalconix/azure-price-scraper/actions"><img src="https://github.com/xfalconix/azure-price-scraper/actions/workflows/main_httptriggerpriceweb.yml/badge.svg" alt="CI"/></a>

Azure Function con **HTTP Trigger** que hace web scraping de precios de productos: extrae nombre y precio de páginas compatibles con los marcadores HTML del código y devuelve la información en texto plano con timestamp.

---

## 🎯 Objetivo

Practicar peticiones HTTP, extracción de datos de HTML y respuestas mediante Azure Functions. El alcance actual es una consulta puntual de un producto; el histórico y las alertas quedan como posibles ampliaciones.

---

## ⚡ Arquitectura

```
Petición HTTP (GET)
       │
       ▼
Azure Functions — HTTP Trigger
       │
       ▼
requests.get(URL)  →  str.find() + text slicing
       │
       ▼
Respuesta: nombre + precio + timestamp
```

---

## 🛠 Stack Tecnológico

| Componente | Tecnología |
|-----------|------------|
| Serverless | Azure Functions (Python) |
| HTTP Client | `requests` |
| Despliegue configurado | GitHub Actions |
| Python del workflow | 3.11 |

---

## 📡 Uso de la API

### Endpoint

```
GET https://<your-function-app>.azurewebsites.net/api/http_trigger_precios_web
```

### Parámetros

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| `url` | query string | URL del producto a consultar (opcional; utiliza la URL de ejemplo del código si se omite) |

### Ejemplo de respuesta

Formato ilustrativo; los valores no representan una consulta actual.

```
Producto:   Válvula de bola DN50 3-piece SS316
Precio:     89,50 €
Timestamp:  2026-04-15 10:32:01
URL:        https://www.aftgrupo.com/productos/ref-14062510
```

### Ejemplo de llamada

```bash
curl "https://<your-function-app>.azurewebsites.net/api/http_trigger_precios_web?url=https://www.aftgrupo.com/productos/ref-14062510"
```

---

## 📁 Estructura del proyecto

```
.
├── function_app.py              # Azure Function with HTTP trigger
├── host.json                    # Azure Functions runtime configuration
├── requirements.txt             # Python dependencies
├── .funcignore                  # Deployment exclusions
├── .github/workflows/           # GitHub Actions deployment workflow
│   └── main_httptriggerpriceweb.yml
└── README.md
```

---

## 🚀 Despliegue

### Requisitos previos

- Cuenta de Azure con Functions habilitado
- Azure CLI instalado (`az login`)
- Azure Functions Core Tools

El repositorio incluye un workflow de despliegue a Azure. Su configuración no garantiza que el endpoint esté activo actualmente.

Antes de desplegar, revisar cuenta, recursos, credenciales y dependencias. El código acepta URL sin restringir destinos y declara acceso anónimo: revisar estos puntos antes de exponerlo públicamente.

**Nota:** publicar cambios en `main` puede activar el workflow de despliegue.

---

## 📊 Desarrollo local

Requiere Python y Azure Functions Core Tools. `logging` y `datetime` pertenecen a la biblioteca estándar; no necesitan instalación como paquetes externos, aunque figuren en el `requirements.txt` actual. Utiliza un entorno virtual para instalar las dependencias.

```bash
# Install the external packages used by the function
python -m pip install azure-functions requests

# Start the local runtime
func start

# Request the product URL configured in the code
curl "http://localhost:7071/api/http_trigger_precios_web"
```

---

## 🔧 Personalización

Para apuntar a una web diferente, modifica los tags HTML en `function_app.py`:

```python
start_name_tag = '<strong>DESCRIPCIÓN:</strong><br>'
start_price_tag = '<div class="precio_ficha">'
```

Inspecciona el HTML del sitio destino con DevTools (F12) para ajustar los marcadores. Si no aparecen, el campo devuelve `No encontrado`. El extractor no ejecuta JavaScript.

---

## 📌 Mejoras futuras

- [ ] Extracción de múltiples productos en una sola llamada
- [ ] Almacenamiento de histórico en Azure Cosmos DB o Blob Storage
- [ ] Alertas por email o Telegram cuando el precio baje
- [ ] Despliegue con Terraform + infraestructura como código
- [ ] Tests con Playwright para validar extracción en sitios dinámicos (JS-rendered)

---

## 👤 Autor

Carlos Falconi — Project Engineer transicionando a Data & MLOps  
📍 ESESA Business School, Málaga — 2025-2026

Ejercicio desarrollado durante clases guiadas y adaptado al portafolio.
