# Desarrollo de APIs REST con Python y Flask – Guía Práctica  
**Autor:** Sergio Daniel Serbluk  
**Analista de Sistemas**  
**Profesor de Nivel Secundario – Modalidad Técnico Profesional**

---

## Nivel 1 – Postman y APIs Públicas
**Objetivo:** Afianzar fundamentos HTTP usando Postman con APIs públicas. Practicamos Params, Headers, Body y Tests.

> **Revisa antes:** Marco Teórico, Nivel 1. En especial las secciones 1.3 (estructura de la solicitud), 1.4 (métodos) y 1.5 (códigos de estado). Tendrás que aplicar exactamente esos conceptos para justificar cada paso en Postman.

### 1.1 JSONPlaceholder – Ejercicios  
- **GET /posts?userId=1** → listar publicaciones del usuario 1.  
- **GET /users/1** → obtener usuario 1.  
- **POST /posts** (Body: raw JSON) → crear recurso (verifica Status 201).
- **PUT /posts/1** → reemplazar título (practicar idempotencia).
- **DELETE /posts/1** → borrar (esperar 200/204).

Antes de enviar cada petición:
- Contrasta el método elegido con la tabla de verbos de la sección 1.4 del marco teórico.
- Comprueba que los headers `Content-Type` y `Accept` respetan lo indicado en 1.3.
- Anota el código de estado recibido y compáralo con la guía de 1.5. Si no coincide, describe por qué la API responde distinto (por ejemplo, JSONPlaceholder permite deletes ficticios y devuelve `200`).

**Sugerencia de Tests en Postman (pestaña Tests):**
```javascript
pm.test('Status 200/201', ()=> pm.expect(pm.response.code).to.be.oneOf([200,201]));
pm.test('JSON válido', ()=> pm.response.to.be.json);
```

---

### 1.2 The Dog API – Imágenes Aleatorias
**GET /v1/images/search?limit=3** → practicar *query params* y analizar headers de respuesta.

- Identifica los encabezados de caché y compáralos con las buenas prácticas de la sección 1.7.
- Guarda al menos una URL de imagen y explica qué recurso está representando (sección 1.1).

---

### 1.3 OpenWeather – v2.5 (gratuita) y v3.0 One Call
- **v2.5:**  
  ```
  GET /data/2.5/weather?q=Buenos Aires&appid=API_KEY&units=metric&lang=es
  ```  
- **v3.0 One Call:** requiere suscripción adicional; primero obtener *lat/lon* con `/geo/1.0/direct`.

**Diagnóstico de errores comunes:**
`401` (API key inválida o sin suscripción), `404` (ruta mal escrita), `429` (límite de peticiones).

> **Referencia teórica:** ver *Marco Teórico, Nivel 1 (HTTP/JSON)*.

**Checklist antes de avanzar al Nivel 2:**
- ¿Puedes explicar qué parte del contrato REST se violó cuando aparece cada código de error? Usa como guía la sección 1.5.
- ¿Identificaste qué parámetros deben viajar en la URL y cuáles en el body? Refiere a la sección 1.3.
- ¿Registraste ejemplos de respuestas JSON bien formadas para reutilizarlas en la automatización? Consulta la sección 1.6.

---

## Nivel 2 – Consumo con Python (`requests` + `.env`)
**Objetivo:** Replicar desde Python lo trabajado en Postman usando `requests`, `.env` y manejo de errores.

> **Revisa antes:** Marco Teórico, Nivel 2 (secciones 2.1 a 2.5). Allí se detalla por qué `requests` necesita ciertos parámetros y cómo estructurar el manejo de excepciones.

### 2.1 Setup del Entorno  
Instalar dependencias:  
```bash
pip install requests python-dotenv
```
Crear archivo `.env` con claves (cuando aplique).

---

### 2.2 Ejercicios Guiados

#### Ejercicio A – GET con filtros (JSONPlaceholder)
```python
import requests
r = requests.get('https://jsonplaceholder.typicode.com/comments', params={'postId':1}, timeout=5)
print(r.status_code)
print(r.json()[:3])
```

- Relaciona la línea `params={'postId':1}` con el análisis de query params del Nivel 1 y con la descripción de la función `requests.get` del punto 2.1 del marco teórico.
- Evalúa si el `timeout=5` es suficiente según los criterios de la sección 2.3. Ajusta y justifica.

#### Ejercicio B – POST / PUT / DELETE (simular CRUD)
```python
import requests
base = 'https://jsonplaceholder.typicode.com/posts'

# POST
nuevo = {'title':'Demo', 'body':'Contenido', 'userId':1}
creado = requests.post(base, json=nuevo, timeout=5)
print(creado.status_code, creado.json())

# PUT
pid = 1
mod = requests.put(f'{base}/{pid}', json={'id':pid,'title':'Mod','body':'x','userId':1})
print(mod.status_code)

# DELETE
borr = requests.delete(f'{base}/{pid}')
print(borr.status_code)
```

- Antes de cada operación, repasa el checklist de idempotencia y códigos de estado (secciones 1.4, 1.5 y 2.3 del marco teórico).
- Documenta en comentarios qué cambio esperarías en una API real y compáralo con el comportamiento simulado de JSONPlaceholder.

#### Ejercicio C – OpenWeather con `.env` y manejo de errores
```python
import os, requests
from dotenv import load_dotenv

load_dotenv()
API = os.getenv('API_KEY')
params = {'q':'Buenos Aires','appid':API,'units':'metric','lang':'es'}

try:
    r = requests.get('https://api.openweathermap.org/data/2.5/weather', params=params, timeout=8)
    r.raise_for_status()
    data = r.json()
    print(data['name'], data['main']['temp'])
except requests.exceptions.RequestException as e:
    print('Error de red/API:', e)
```

- Verifica que `.env` contenga la `API_KEY` y que esté cargada siguiendo la guía de la sección 2.4.
- Añade logs o `print` informativos que indiquen qué endpoint se invocó y cuál fue la razón del error, como se sugiere en la sección 2.3.
- Si obtienes un código `429`, documenta el tiempo de espera recomendado por el header `Retry-After`.

---

### 2.3 Desafíos (para nota)
- Guardar respuestas en JSON/CSV y listar los 5 primeros resultados.
- Crear una función `get_posts(user_id)` con manejo de errores y timeout.
- Escribir *asserts* (o tests en Postman) que validen status y campos clave.

> **Referencias teóricas:** ver *Marco Teórico, Nivel 1 (HTTP/JSON)* y *Nivel 2 (.env, manejo de errores)*.

> **Sugerencia:** resuelve primero los ejercicios guiados y deja un registro (códigos de estado, ejemplos de payloads) en un cuaderno compartido. Esa evidencia será tu material de consulta cuando implementes la API en Flask (Marco Teórico, Nivel 3) y necesites validar manualmente cada endpoint.
