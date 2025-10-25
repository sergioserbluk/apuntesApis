# Desarrollo de APIs REST con Python y Flask – Marco Teórico
**Autor:** Sergio Daniel Serbluk  
**Analista de Sistemas**  
**Profesor de Nivel Secundario – Modalidad Técnico Profesional**

---

## Nivel 1 – Conceptos Fundamentales  
**Objetivo:** Comprender qué es una API, cómo se comunican los sistemas por HTTP y cómo se representan los datos en JSON.

### 1.1 ¿Qué es una API?
Una **API (Application Programming Interface)** es un contrato para que un cliente consuma recursos de un servidor.
Define **endpoints**, **métodos**, **entradas** (params, headers, body) y **salidas** (status, headers, body).

En términos de arquitectura de software, una API abstrae la lógica interna del sistema y expone únicamente una fachada bien definida. Esta capa de interacción reduce el acoplamiento entre componentes, facilita la reutilización de servicios y permite evolucionar internamente sin romper a los consumidores si se mantiene la compatibilidad del contrato. Las APIs REST se apoyan en el protocolo HTTP y en el modelo de recursos para representar el estado del sistema mediante identificadores únicos (URLs) y representaciones en distintos formatos, principalmente JSON.

---

### 1.2 Modelo Cliente–Servidor
- **Cliente:** navegador, Postman, script Python, app móvil.
- **Servidor:** servicio web que expone la API.
- **Intercambio:** solicitudes (*request*) y respuestas (*response*) por HTTP/HTTPS.

El patrón cliente–servidor separa responsabilidades: el cliente gestiona la interfaz de usuario o el consumo automatizado, mientras el servidor concentra la lógica de negocio y el acceso a datos. Esta separación favorece la escalabilidad horizontal (multiplicar clientes sin alterar el servidor), mejora la mantenibilidad y permite emplear distintas tecnologías en cada extremo. HTTP actúa como protocolo de aplicación stateless, por lo que cada petición debe contener toda la información necesaria para ser atendida, evitando dependencias con peticiones previas.

---

### 1.3 Elementos de una Solicitud HTTP
- **Método:** GET / POST / PUT / PATCH / DELETE
- **URL:** esquema + host + ruta + query params  
- **Headers:** metadatos (Content-Type, Accept, Authorization)
- **Body:** datos (JSON) para crear o actualizar

Una solicitud HTTP está compuesta por la línea de petición (método + ruta + versión), encabezados que describen aspectos de la comunicación (tipo de contenido, longitud, autenticación) y un cuerpo opcional. La comprensión de estos elementos es esencial para diagnosticar problemas y diseñar interfaces robustas. Por ejemplo, el header `Accept` indica el formato de respuesta esperado, mientras que `Content-Type` declara el formato de los datos enviados. En APIs modernas es habitual emplear `application/json` y soportar compresión (`Accept-Encoding`) para optimizar el intercambio.

---

### 1.4 Métodos HTTP
| Método | Descripción |
|--------|--------------|
| GET | Leer sin modificar |
| POST | Crear un recurso nuevo |
| PUT | Reemplazo total |
| PATCH | Actualización parcial |
| DELETE | Eliminar recurso |

Los métodos HTTP, también llamados verbos, transmiten la intención de la operación. REST promueve el uso semántico de los verbos estándar para aprovechar las funcionalidades del protocolo (caché en GET, idempotencia en PUT/DELETE, etc.). Además, existen métodos menos utilizados como `HEAD` (obtener headers sin cuerpo), `OPTIONS` (descubrir métodos permitidos) o `TRACE`. Los desarrolladores deben diseñar los endpoints respetando estas convenciones para garantizar interoperabilidad con proxies, herramientas de depuración y clientes reutilizables.

---

### 1.5 Códigos de Estado
`200 OK`, `201 Created`, `204 No Content`
`400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`
`409 Conflict`, `422 Unprocessable Entity`, `500 Internal Server Error`

Los códigos de estado HTTP están agrupados en clases (1xx informativos, 2xx éxito, 3xx redirecciones, 4xx errores del cliente, 5xx errores del servidor). Proporcionar códigos correctos aumenta la transparencia de la API y facilita la automatización. Por ejemplo, `201 Created` debe acompañarse de un header `Location` que apunte al nuevo recurso. Del mismo modo, `409 Conflict` comunica colisiones de datos (duplicados, conflictos de versión) y `422` indica errores semánticos en la validación, diferenciando ambos escenarios de un simple `400`.

---

### 1.6 JSON en APIs
JSON es texto estructurado **clave–valor**, fácil de leer y parsear.

```json
{ "id": 1, "titulo": "Aprender Flask", "hecha": false }
```

**Buenas prácticas:** usar *snake_case* o *camelCase* consistente; validar esquemas.

JSON deriva de la notación literal de objetos de JavaScript pero es un formato independiente del lenguaje. Admite estructuras anidadas mediante arreglos y objetos, lo que facilita representar documentos complejos. Para asegurar la calidad de los datos, se recomienda definir esquemas (por ejemplo con JSON Schema) que especifiquen tipos, rangos y campos obligatorios. Asimismo, es importante evitar ambigüedades en el tratamiento de números (enteros vs. flotantes) y fechas, empleando formatos estándar como ISO 8601.

---

### 1.7 Buenas Prácticas de Diseño REST
- Rutas en plural orientadas a recursos: `/api/v1/tareas`
- Versionado: `/api/v1/...`
- Idempotencia en PUT/DELETE
- Respuestas coherentes: siempre JSON + status apropiado

El diseño RESTful se basa en la identificación de recursos (entidades manipulables) y en la representación consistente de sus estados. Se recomienda estructurar las rutas jerárquicamente (`/usuarios/42/tareas`) para reflejar relaciones y emplear filtros mediante parámetros de consulta (`?estado=pendiente`). El versionado explícito evita romper integraciones existentes al introducir cambios incompatibles. Finalmente, es conveniente documentar reglas de paginación, ordenamiento y enlaces HATEOAS cuando se quiera guiar al cliente a través de flujos complejos.

> **Referencia práctica:** ver Guía práctica, Nivel 1 (Postman y APIs públicas).

---

## Nivel 2 – Consumo de APIs desde Python  
**Objetivo:** replicar en código Python lo probado en Postman, usando la librería `requests`, manejo de errores y variables de entorno (.env).

### 2.1 Librería `requests`
```python
requests.get/post/put/delete(url, params=..., json=..., headers=...)
response.status_code
response.json()
response.headers
response.text
r.raise_for_status()
```

`requests` abstrae detalles de sockets y manejo manual del protocolo HTTP, ofreciendo una API Pythonica. Permite añadir autenticación (Basic, Token, Bearer), gestionar sesiones persistentes con `requests.Session()` para reutilizar conexiones TCP (mejorando performance) y configurar adaptadores personalizados para estrategias de reintento (`Retry`). Comprender estas capacidades resulta útil al construir clientes robustos para APIs reales.

---

### 2.2 Ejemplos Típicos

**GET con parámetros:**
```python
import requests
r = requests.get('https://jsonplaceholder.typicode.com/posts', params={'userId': 1})
print(r.status_code)
print(r.json()[:2])
```

**POST enviando JSON:**
```python
import requests
nuevo = {'title':'Tarea nueva', 'body':'Practicar Flask', 'userId':2}
r = requests.post('https://jsonplaceholder.typicode.com/posts', json=nuevo, timeout=5)
r.raise_for_status()
print(r.status_code, r.json())
```

Los ejemplos ilustran la separación entre parámetros de consulta (`params`) y el cuerpo JSON (`json`). En el primer caso, la librería codifica el diccionario en la URL, útil para filtros y paginación. En el segundo, se serializa automáticamente como JSON y se define el header `Content-Type`. Al consumir APIs reales es recomendable envolver las llamadas en funciones reutilizables, manejar reintentos con exponencial backoff y registrar las respuestas para auditorías o trazabilidad.

---

### 2.3 Manejo de Errores y Timeouts
- Usar `timeout` para evitar bloqueos indefinidos.
- Capturar `requests.exceptions.RequestException`.
- Verificar códigos y mensajes del servidor.

Los timeouts deben definirse de acuerdo con la criticidad del proceso y pueden especificarse separadamente para conexión y lectura (`timeout=(3.05, 27)`). El manejo de excepciones debe diferenciar entre problemas de red (`ConnectionError`), de SSL (`SSLError`) o de tiempo (`Timeout`) para tomar acciones específicas (reintentos, degradación controlada). Revisar el contenido de respuesta (`response.json()` o `response.text`) permite construir mensajes de error más descriptivos hacia la capa de presentación.

---

### 2.4 Variables de Entorno y `.env`
Nunca subas claves a repositorios.
Usa `.env` y `python-dotenv` para cargarlas.

```bash
# .env
API_KEY=tu_api_key
```

```python
# app.py
import os
from dotenv import load_dotenv
load_dotenv()
API_KEY = os.getenv('API_KEY')
```

Las variables de entorno representan la forma estándar de parametrizar aplicaciones en distintas etapas (desarrollo, staging, producción) sin modificar el código. Es fundamental establecer valores por defecto seguros o fallar explícitamente si faltan. Además, conviene configurar un archivo `.env.example` con la lista de variables esperadas para facilitar la puesta en marcha de nuevos desarrolladores. En entornos productivos, estas variables suelen gestionarse mediante servicios de secretos (Vault, AWS Secrets Manager) o configuraciones del orquestador (Docker, Kubernetes).

---

### 2.5 Guardado y Parsing de Respuestas
Guardar en JSON/CSV según necesidad; sanitizar y validar datos.

```python
import json
with open('datos.json','w',encoding='utf-8') as f:
    json.dump(r.json(), f, ensure_ascii=False, indent=2)
```

La persistencia local de respuestas facilita la depuración, la generación de reportes y la construcción de datasets para análisis posteriores. Es recomendable definir esquemas consistentes y normalizar datos (por ejemplo, convertir fechas a zona horaria estándar). Para volúmenes grandes, se pueden emplear formatos columnar (Parquet) o bases de datos orientadas a documentos. El parsing también puede incluir validaciones contra contratos definidos con `pydantic` o `marshmallow` para garantizar integridad.

> **Referencia práctica:** ver Guía práctica, Nivel 2 (requests + .env + ejercicios).

---

## Nivel 3 – Creación de APIs con Flask  
**Objetivo:** diseñar y comprender una API REST básica con Flask, cubriendo rutas, decoradores, modelo de datos simple, validaciones, manejo de errores y persistencia mínima.

---

### 3.1 ¿Qué es Flask?
Microframework ligero y extensible, basado en **Werkzeug** (servidor/WSGI) y **Jinja2** (plantillas).
Ideal para aprender REST: rutas explícitas + respuestas JSON.

Flask sigue la filosofía *micro* al proveer solo el núcleo necesario (enrutamiento, contexto de aplicación, sistema de extensiones) y delegar funcionalidades avanzadas a librerías externas. Esto brinda flexibilidad para elegir componentes según las necesidades del proyecto (ORM, validadores, autenticación). Su naturaleza síncrona se ejecuta sobre WSGI, aunque existen extensiones para integrar ASGI y tareas asíncronas mediante Celery o RQ. Comprender el ciclo de vida de la aplicación y los contextos de solicitud es clave para evitar errores de diseño.

---

### 3.2 Decoradores de Ruta y Flujo de Petición
```python
@app.get('/tareas')  
@app.post('/tareas')  
@app.put('/tareas/<int:id>')  
@app.delete('/tareas/<int:id>')
```

**Manejo de errores:**
```python
@app.errorhandler(404)
def not_found(e):
    return {'error': 'Ruta no encontrada'}, 404
```

Cada decorador de ruta asocia un patrón URL con una función vista. Flask transforma la petición en un objeto `Request` accesible a través de `flask.request`, ejecuta la función y convierte la respuesta (diccionarios, tuplas, objetos `Response`) en una salida HTTP. Los parámetros dinámicos (`<int:id>`) incluyen validaciones de tipo básicas y facilitan la construcción de rutas significativas. Implementar manejadores de error centralizados permite responder con JSON uniforme en toda la API y registrar eventos críticos.

---

### 3.3 Estructura Mínima de Proyecto
```
api_tareas/
├─ app.py
├─ storage.json
└─ requirements.txt
```

- `app.py`: inicializa Flask y define endpoints.
- `storage.json`: persistencia simple.
- `requirements.txt`: dependencias.

Aunque este esquema es suficiente para proyectos educativos, en aplicaciones reales conviene separar la configuración (`config.py`), la definición del modelo (`models.py`), los servicios (`services/`) y los controladores (`routes/`). También es común emplear *blueprints* para modularizar funcionalidades (por ejemplo, `auth`, `tareas`). Mantener una estructura coherente facilita la escalabilidad del código y la incorporación de nuevas funcionalidades.

---

### 3.4 Modelo de Recurso  
```json
{ "id": 1, "titulo": "Aprender Flask", "hecha": false }
```

Validaciones: campos obligatorios, tipos correctos.
Rutas RESTful: `/tareas`, `/tareas/<id>`.

Un recurso representa la entidad lógica que se manipula. El modelo debe definir atributos, restricciones y valores por defecto. En APIs educativas puede almacenarse en memoria o en archivos JSON, pero en entornos productivos se recurre a bases de datos relacionales o NoSQL según la naturaleza de los datos. Mantener un identificador inmutable (`id`) simplifica la sincronización con clientes y el manejo de referencias cruzadas.

---

### 3.5 Validación de Entrada
```python
data = request.get_json(silent=True) or {}
titulo = (data.get('titulo') or '').strip()
if len(titulo) < 3:
    return {'error': 'titulo mínimo 3 caracteres'}, 400
hecha = bool(data.get('hecha', False))
```

La validación es el primer escudo contra datos corruptos o maliciosos. Debe incluir verificación de tipos, rangos, longitud y reglas de negocio específicas (unicidad, consistencia temporal). Además de las validaciones manuales, se pueden utilizar bibliotecas como `marshmallow` o `pydantic` para describir esquemas y automatizar la serialización/deserialización. Es buena práctica devolver mensajes claros y localizables para que los clientes puedan corregir la entrada.

---

### 3.6 Manejo de Errores y Códigos HTTP  
```python
@app.errorhandler(405)
def method_not_allowed(e):
    return {'error': 'Método no permitido'}, 405
```

Implementar un *error handling* uniforme evita duplicar lógica en cada endpoint y permite registrar métricas de fallos. Es recomendable capturar excepciones de bajo nivel (por ejemplo, acceso a archivos o base de datos) y convertirlas en respuestas HTTP adecuadas. Asimismo, se pueden definir errores personalizados para expresar conflictos de dominio (limite de cuotas, estados inválidos) sin perder trazabilidad.

---

### 3.7 Persistencia Simple
```python
import json, os
STORAGE='storage.json'

def cargar():
    if os.path.exists(STORAGE):
        with open(STORAGE,'r',encoding='utf-8') as f:
            return json.load(f)
    return []

def guardar(tareas):
    with open(STORAGE,'w',encoding='utf-8') as f:
        json.dump(tareas, f, ensure_ascii=False, indent=2)
```

> **Referencia práctica:** ver Guía práctica, Nivel 3 (CRUD completo y pruebas en Postman).

Esta estrategia de persistencia es adecuada para prototipos y demostraciones, pero no soporta concurrencia ni escalado. Es importante implementar bloqueos o versiones para evitar pérdidas de datos si múltiples procesos escriben simultáneamente. En escenarios reales, se debe migrar a bases de datos transaccionales o sistemas de almacenamiento persistente con transacciones y mecanismos de recuperación.

---

## Nivel 4 – Seguridad, CORS, Persistencia y Documentación  
**Objetivo:** agregar autenticación con JWT, control CORS, persistencia con BD y documentación Swagger.

---

### 4.1 Persistencia con Base de Datos (SQLAlchemy)
```python
from flask_sqlalchemy import SQLAlchemy
app.config['SQLALCHEMY_DATABASE_URI']='sqlite:///tareas.db'
db = SQLAlchemy(app)
```

```python
class Tarea(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    titulo = db.Column(db.String(120), nullable=False)
    hecha = db.Column(db.Boolean, default=False)
```

SQLAlchemy es un ORM (Object Relational Mapper) que permite mapear clases de Python a tablas de base de datos. Ofrece una capa de abstracción que simplifica las operaciones CRUD y provee un motor de consultas expresivo. Para proyectos más grandes se recomienda separar el modelo declarativo, gestionar sesiones con `scoped_session` y aplicar migraciones con `Flask-Migrate`. Elegir el motor adecuado (SQLite, PostgreSQL, MySQL) dependerá de requerimientos de concurrencia, escalabilidad y soporte transaccional.

---

### 4.2 Autenticación con JWT
```python
from flask_jwt_extended import JWTManager, create_access_token
app.config['JWT_SECRET_KEY']='cambia-esta-clave'
jwt = JWTManager(app)
```

```python
@app.post('/login')
def login():
    token = create_access_token(identity='usuario_demo')
    return {'access_token': token}
```

JSON Web Tokens encapsulan información de identidad y claims en un objeto firmado digitalmente. Se componen de tres partes (header, payload, firma) codificadas en Base64URL. En APIs REST se utilizan como *bearer tokens* enviados en el header `Authorization`. Es esencial establecer tiempos de expiración (`expires_delta`), rotar claves secretas y considerar mecanismos de *refresh tokens* para mantener sesiones seguras. Para escenarios críticos, es conveniente almacenar tokens revocados o implementar *token blacklisting*.

---

### 4.3 CORS (Cross-Origin Resource Sharing)
```python
from flask_cors import CORS
CORS(app)
# o restringido
# CORS(app, resources={r"/api/*": {"origins": ["https://tu-frontend.com"]}})
```

CORS regula cómo un navegador permite que un script ejecutado en un origen (dominio + puerto + protocolo) acceda a recursos alojados en otro. Flask-CORS añade los headers necesarios (`Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, etc.) para evitar bloqueos por política de mismo origen. En producción se debe limitar la lista de orígenes confiables y habilitar credenciales solo cuando sea imprescindible, mitigando riesgos de ataques CSRF o filtrado de información.

---

### 4.4 Documentación con Swagger
```python
from flasgger import Swagger
Swagger(app)  # /apidocs
```

Swagger (OpenAPI) proporciona un contrato legible tanto por humanos como por máquinas que describe endpoints, parámetros, esquemas y respuestas. Documentar la API facilita el onboarding de nuevos desarrolladores, habilita la generación automática de clientes SDK y sirve como base para pruebas automatizadas. Mantener la documentación sincronizada con el código requiere integrar herramientas de validación continua que detecten divergencias.

---

### 4.5 Manejo de Errores
```python
@app.errorhandler(404)
def not_found(e):
    return {'error':'Ruta no encontrada'}, 404
```

Además de los códigos genéricos, es recomendable definir una estructura estándar para las respuestas de error (`code`, `message`, `details`) y registrar cada incidente con información contextual (endpoint, usuario, payload). Esto facilita la observabilidad y el análisis post-mortem. Integrar sistemas de monitoreo (Sentry, Elastic APM) permite detectar patrones y priorizar correcciones.

---

### 4.6 Buenas Prácticas de Seguridad
- Variables en `.env` (JWT_SECRET_KEY, DB URI).
- Rate limiting y logs.
- Validación estricta y sanitización.

> **Referencia práctica:** Guía práctica, Nivel 4 (CORS + JWT + BD + Swagger).

La seguridad debe abordarse en capas: autenticación robusta, autorización basada en roles o scopes, protección contra inyección (sanitizar entradas, usar consultas parametrizadas), cifrado en tránsito (HTTPS) y en reposo cuando sea necesario. También conviene implementar monitoreo de intentos fallidos, políticas de contraseñas y análisis de vulnerabilidades (OWASP Top 10) para mantener la API protegida frente a ataques comunes.

---

## Nivel 5 – Despliegue, Observabilidad y Buenas Prácticas  
**Objetivo:** preparar la API para producción.

---

### 5.1 Configuración por Entorno
```bash
FLASK_ENV=production
DEBUG=False
LOG_LEVEL=INFO
```

Separar configuraciones por entorno permite ajustar parámetros sensibles sin modificar el código fuente. Flask utiliza objetos de configuración que pueden derivarse de clases (`Config`, `DevelopmentConfig`, `ProductionConfig`). Esto habilita activar/desactivar extensiones, cambiar URIs de base de datos y modificar políticas de caché según el contexto. Una buena práctica es centralizar la carga de configuración y validar que todos los valores críticos estén presentes al iniciar la aplicación.

---

### 5.2 Servidor WSGI
```bash
pip install gunicorn
gunicorn app:app -w 2 -k gthread -b 0.0.0.0:8000
```

Gunicorn actúa como servidor WSGI de producción, gestionando múltiples procesos y workers. Configurar correctamente el número de workers (`-w`) y el tipo (`sync`, `gthread`, `gevent`) depende del patrón de carga y del uso de I/O. Para aplicaciones CPU-bound se recomienda emplear más procesos, mientras que para cargas I/O-bound se pueden usar workers asíncronos. Integrarlo con un proxy inverso (Nginx) mejora el manejo de TLS, compresión y balanceo.

---

### 5.3 Despliegue en la Nube
- **PaaS:** Render, Railway, Fly.io
- **Docker:** portabilidad
- **CDN/Proxy:** TLS y caching

La estrategia de despliegue debe considerar automatización, escalabilidad y monitoreo. Contenerizar la aplicación con Docker facilita la reproducibilidad entre entornos y permite ejecutar la misma imagen en distintas plataformas. Servicios PaaS simplifican la gestión de infraestructura, mientras que en arquitecturas avanzadas se puede optar por Kubernetes para orquestar múltiples instancias, escalar automáticamente y gestionar configuraciones centralizadas.

---

### 5.4 Seguridad en Producción
- HTTPS obligatorio
- CORS restringido
- Headers de seguridad
- Backups y rotación de claves

Implementar HTTPS con certificados válidos (Let's Encrypt, proveedores comerciales) protege la confidencialidad e integridad de los datos en tránsito. Los headers de seguridad (`Strict-Transport-Security`, `Content-Security-Policy`, `X-Content-Type-Options`) reducen superficies de ataque en clientes web. Los backups periódicos y la rotación de claves son esenciales para resiliencia y cumplimiento normativo.

---

### 5.5 Observabilidad
```python
import logging
logging.basicConfig(level='INFO', format='%(asctime)s %(levelname)s %(message)s')
app.logger.info('API iniciada')
```

La observabilidad combina métricas, logs y trazas distribuidas para entender el comportamiento de la aplicación. Integrar herramientas como Prometheus (métricas), Grafana (dashboards) y OpenTelemetry (trazas) permite detectar cuellos de botella y diagnosticar errores. Los logs deben incluir correlación de request IDs para rastrear el recorrido de una petición a través de microservicios.

---

### 5.6 Migraciones y Datos
Usar **Alembic/Flask-Migrate** para cambios de esquema sin pérdida.
Crear *scripts de seed* y plan de rollback.

Las migraciones controlan la evolución del esquema de la base de datos. Deben ejecutarse como parte del proceso de despliegue para mantener sincronizados los ambientes. Además, se recomienda versionar los datos iniciales (semillas) y documentar estrategias de reversión ante errores. Herramientas como Alembic generan scripts incrementales que pueden revisarse y personalizarse antes de aplicarse en producción.

---

### 5.7 CI/CD
```yaml
name: ci
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with: { python-version: '3.11' }
      - run: pip install -r requirements.txt
      - run: pytest -q
```

Los pipelines de integración continua automatizan pruebas, linting y análisis estático en cada cambio, reduciendo el riesgo de introducir regresiones. La entrega continua añade etapas de despliegue automatizado, usualmente con gates manuales o pruebas de smoke. Es crucial proteger las ramas principales, usar revisiones de código y almacenar artefactos generados para trazabilidad.

---

### 5.8 Estrategias de Escalado
- Escalado horizontal o vertical
- Cache
- Colas (RQ/Celery)

El escalado vertical aumenta recursos de una instancia (CPU, memoria), mientras que el horizontal multiplica réplicas detrás de un balanceador. Las caches (Redis, Memcached) reducen carga en la base de datos almacenando resultados temporales. Las colas de tareas delegan trabajos pesados o diferidos (envío de emails, procesamiento de archivos) a workers especializados, desacoplando el tiempo de respuesta del endpoint.

---

### 5.9 Checklist de Producción
✅ `DEBUG=False`
✅ `SECRET_KEY` seguro
✅ `CORS` restringido
✅ Gunicorn activo
✅ Logs y métricas visibles
✅ Backups configurados

> **Referencia práctica:** Guía práctica, Nivel 5 (Deploy en Render/Railway, Docker, smoke tests y rollback).

Este checklist debe complementarse con monitoreo de salud (endpoints `/health`), pruebas automatizadas post-despliegue y planes de contingencia documentados. Revisar periódicamente la configuración asegura que no se introduzcan regresiones de seguridad o rendimiento cuando la aplicación evoluciona.
