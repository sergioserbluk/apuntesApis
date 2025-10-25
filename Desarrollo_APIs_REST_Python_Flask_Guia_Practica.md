# Desarrollo de APIs REST con Python y Flask – Guía Práctica  
**Autor:** Sergio Daniel Serbluk  
**Analista de Sistemas**  
**Profesor de Nivel Secundario – Modalidad Técnico Profesional**

---

## Nivel 1 – Postman y APIs Públicas  
**Objetivo:** Afianzar fundamentos HTTP usando Postman con APIs públicas. Practicamos Params, Headers, Body y Tests.

### 1.1 JSONPlaceholder – Ejercicios  
- **GET /posts?userId=1** → listar publicaciones del usuario 1.  
- **GET /users/1** → obtener usuario 1.  
- **POST /posts** (Body: raw JSON) → crear recurso (verifica Status 201).  
- **PUT /posts/1** → reemplazar título (practicar idempotencia).  
- **DELETE /posts/1** → borrar (esperar 200/204).  

**Sugerencia de Tests en Postman (pestaña Tests):**
```javascript
pm.test('Status 200/201', ()=> pm.expect(pm.response.code).to.be.oneOf([200,201]));
pm.test('JSON válido', ()=> pm.response.to.be.json);
```

---

### 1.2 The Dog API – Imágenes Aleatorias  
**GET /v1/images/search?limit=3** → practicar *query params* y analizar headers de respuesta.

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

---

## Nivel 2 – Consumo con Python (`requests` + `.env`)  
**Objetivo:** Replicar desde Python lo trabajado en Postman usando `requests`, `.env` y manejo de errores.

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

---

### 2.3 Desafíos (para nota)  
- Guardar respuestas en JSON/CSV y listar los 5 primeros resultados.  
- Crear una función `get_posts(user_id)` con manejo de errores y timeout.  
- Escribir *asserts* (o tests en Postman) que validen status y campos clave.

> **Referencias teóricas:** ver *Marco Teórico, Nivel 1 (HTTP/JSON)* y *Nivel 2 (.env, manejo de errores)*.
