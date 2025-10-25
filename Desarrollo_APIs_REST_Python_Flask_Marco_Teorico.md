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

---

### 1.2 Modelo Cliente–Servidor  
- **Cliente:** navegador, Postman, script Python, app móvil.  
- **Servidor:** servicio web que expone la API.  
- **Intercambio:** solicitudes (*request*) y respuestas (*response*) por HTTP/HTTPS.

---

### 1.3 Elementos de una Solicitud HTTP  
- **Método:** GET / POST / PUT / PATCH / DELETE  
- **URL:** esquema + host + ruta + query params  
- **Headers:** metadatos (Content-Type, Accept, Authorization)  
- **Body:** datos (JSON) para crear o actualizar

---

### 1.4 Métodos HTTP  
| Método | Descripción |
|--------|--------------|
| GET | Leer sin modificar |
| POST | Crear un recurso nuevo |
| PUT | Reemplazo total |
| PATCH | Actualización parcial |
| DELETE | Eliminar recurso |

---

### 1.5 Códigos de Estado  
`200 OK`, `201 Created`, `204 No Content`  
`400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`  
`409 Conflict`, `422 Unprocessable Entity`, `500 Internal Server Error`

---

### 1.6 JSON en APIs  
JSON es texto estructurado **clave–valor**, fácil de leer y parsear.

```json
{ "id": 1, "titulo": "Aprender Flask", "hecha": false }
```

**Buenas prácticas:** usar *snake_case* o *camelCase* consistente; validar esquemas.

---

### 1.7 Buenas Prácticas de Diseño REST  
- Rutas en plural orientadas a recursos: `/api/v1/tareas`  
- Versionado: `/api/v1/...`  
- Idempotencia en PUT/DELETE  
- Respuestas coherentes: siempre JSON + status apropiado  

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

---

### 2.3 Manejo de Errores y Timeouts  
- Usar `timeout` para evitar bloqueos indefinidos.  
- Capturar `requests.exceptions.RequestException`.  
- Verificar códigos y mensajes del servidor.

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

---

### 2.5 Guardado y Parsing de Respuestas  
Guardar en JSON/CSV según necesidad; sanitizar y validar datos.

```python
import json
with open('datos.json','w',encoding='utf-8') as f:
    json.dump(r.json(), f, ensure_ascii=False, indent=2)
```

> **Referencia práctica:** ver Guía práctica, Nivel 2 (requests + .env + ejercicios).

---

## Nivel 3 – Creación de APIs con Flask  
**Objetivo:** diseñar y comprender una API REST básica con Flask, cubriendo rutas, decoradores, modelo de datos simple, validaciones, manejo de errores y persistencia mínima.

---

### 3.1 ¿Qué es Flask?  
Microframework ligero y extensible, basado en **Werkzeug** (servidor/WSGI) y **Jinja2** (plantillas).  
Ideal para aprender REST: rutas explícitas + respuestas JSON.

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

---

### 3.4 Modelo de Recurso  
```json
{ "id": 1, "titulo": "Aprender Flask", "hecha": false }
```

Validaciones: campos obligatorios, tipos correctos.  
Rutas RESTful: `/tareas`, `/tareas/<id>`.

---

### 3.5 Validación de Entrada  
```python
data = request.get_json(silent=True) or {}
titulo = (data.get('titulo') or '').strip()
if len(titulo) < 3:
    return {'error': 'titulo mínimo 3 caracteres'}, 400
hecha = bool(data.get('hecha', False))
```

---

### 3.6 Manejo de Errores y Códigos HTTP  
```python
@app.errorhandler(405)
def method_not_allowed(e):
    return {'error': 'Método no permitido'}, 405
```

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

---

### 4.3 CORS (Cross-Origin Resource Sharing)  
```python
from flask_cors import CORS
CORS(app)
# o restringido
# CORS(app, resources={r"/api/*": {"origins": ["https://tu-frontend.com"]}})
```

---

### 4.4 Documentación con Swagger  
```python
from flasgger import Swagger
Swagger(app)  # /apidocs
```

---

### 4.5 Manejo de Errores  
```python
@app.errorhandler(404)
def not_found(e):
    return {'error':'Ruta no encontrada'}, 404
```

---

### 4.6 Buenas Prácticas de Seguridad  
- Variables en `.env` (JWT_SECRET_KEY, DB URI).  
- Rate limiting y logs.  
- Validación estricta y sanitización.

> **Referencia práctica:** Guía práctica, Nivel 4 (CORS + JWT + BD + Swagger).

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

---

### 5.2 Servidor WSGI  
```bash
pip install gunicorn
gunicorn app:app -w 2 -k gthread -b 0.0.0.0:8000
```

---

### 5.3 Despliegue en la Nube  
- **PaaS:** Render, Railway, Fly.io  
- **Docker:** portabilidad  
- **CDN/Proxy:** TLS y caching

---

### 5.4 Seguridad en Producción  
- HTTPS obligatorio  
- CORS restringido  
- Headers de seguridad  
- Backups y rotación de claves  

---

### 5.5 Observabilidad  
```python
import logging
logging.basicConfig(level='INFO', format='%(asctime)s %(levelname)s %(message)s')
app.logger.info('API iniciada')
```

---

### 5.6 Migraciones y Datos  
Usar **Alembic/Flask-Migrate** para cambios de esquema sin pérdida.  
Crear *scripts de seed* y plan de rollback.

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

---

### 5.8 Estrategias de Escalado  
- Escalado horizontal o vertical  
- Cache  
- Colas (RQ/Celery)

---

### 5.9 Checklist de Producción  
✅ `DEBUG=False`  
✅ `SECRET_KEY` seguro  
✅ `CORS` restringido  
✅ Gunicorn activo  
✅ Logs y métricas visibles  
✅ Backups configurados

> **Referencia práctica:** Guía práctica, Nivel 5 (Deploy en Render/Railway, Docker, smoke tests y rollback).
