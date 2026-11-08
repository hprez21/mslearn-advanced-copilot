[![Abrir en GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/MicrosoftDocs/mslearn-advanced-copilot)

**Idiomas:** [English](README.md) | Español

# Aplica técnicas avanzadas de GitHub Copilot

Descubre nuevas formas de aplicar técnicas avanzadas de GitHub Copilot en un repositorio de Python para implementar un formulario HTML interactivo y un endpoint de una interfaz de programación de aplicaciones (API).
Adquiere experiencia práctica con este repositorio, que contiene una aplicación web de Python con una API de información meteorológica para viajes.

[![Con tecnología de Awesome Copilot](https://img.shields.io/badge/Powered_by-Awesome_Copilot-blue?logo=githubcopilot)](https://aka.ms/awesome-github-copilot)

Esta guía traduce el texto explicativo al español. Los comandos, las rutas, los valores de los datos y los prompts originales de Copilot se conservan para que puedas seguir el mismo laboratorio.

## Requisitos

1. Habilita tu [servicio de GitHub Copilot](https://github.com/github-copilot/signup).
1. Abre [este repositorio en Codespaces](https://codespaces.new/MicrosoftDocs/mslearn-copilot-codespaces-python).

### Detalles adicionales de configuración

Se conservan los tres pasos del ejercicio original, incluidos sus atajos de teclado y prompts de Copilot. Las siguientes secciones añaden instrucciones para iniciar la aplicación, enviar solicitudes y comprobar los resultados.

**Aclaración sobre el enlace de Codespaces:** el enlace original de la sección Requisitos apunta a otro repositorio de ejemplo. Para este laboratorio, usa el botón al principio del README o [abre este repositorio en Codespaces](https://codespaces.new/MicrosoftDocs/mslearn-advanced-copilot).

Para trabajar de forma local, instala Git, Python 3.10 o posterior y VS Code con las extensiones de Python y GitHub Copilot. El contenedor de desarrollo usa Python 3.10. La API lee el archivo `weather.json` incluido, por lo que no necesitas una cuenta de un servicio meteorológico, una clave de API ni una base de datos para ejecutarla.

## Configura e inicia la API

Elige una de las siguientes opciones. Ejecuta todos los comandos desde la raíz del repositorio: la carpeta que contiene `main.py` y `requirements.txt`.

### Opción A: GitHub Codespaces

1. [Abre este repositorio en Codespaces](https://codespaces.new/MicrosoftDocs/mslearn-advanced-copilot) y crea un codespace.
2. Espera a que termine la creación del contenedor de desarrollo. La configuración de [`.devcontainer/devcontainer.json`](.devcontainer/devcontainer.json) instala las dependencias e inicia Uvicorn automáticamente.
3. Abre la pestaña **Ports** (Puertos) de VS Code, busca el puerto **8000** y selecciona **Open in Browser** (Abrir en el navegador). Añade `/docs` al final de la URL del puerto si la documentación de la API no se abre automáticamente.
4. Si el servidor no se inició, ejecuta este comando en la terminal:

   ```bash
   /home/vscode/venv/bin/python -m uvicorn main:app --host 0.0.0.0 --port 8000 --reload
   ```

No inicies un segundo servidor si la aplicación ya está funcionando en el puerto 8000. En los comandos de las siguientes secciones, usa `/home/vscode/venv/bin/python` si `python` apunta a otro entorno. En los ejemplos para el navegador, sustituye `http://127.0.0.1:8000` por la URL del puerto de tu codespace; los comandos ejecutados dentro del codespace pueden usar la dirección local.

### Opción B: Ejecución local

Clona el repositorio y entra en su carpeta. Si ya tienes una copia, abre una terminal en esa carpeta y omite este bloque de comandos.

```bash
git clone https://github.com/MicrosoftDocs/mslearn-advanced-copilot.git
cd mslearn-advanced-copilot
```

Crea y activa un entorno virtual con los comandos correspondientes a tu shell.

**Windows PowerShell:**

```powershell
python --version
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**macOS / Linux (Bash o Zsh):**

```bash
python3 --version
python3 -m venv .venv
source .venv/bin/activate
```

Con el entorno virtual activo, instala las dependencias e inicia el servidor:

```bash
python -m pip install -r requirements.txt
python -m uvicorn main:app --host 127.0.0.1 --port 8000 --reload
```

Espera a que aparezca `Application startup complete` y abre **http://127.0.0.1:8000/docs** en el navegador. Mantén esta terminal en ejecución y usa una segunda terminal para enviar solicitudes y ejecutar pruebas. La opción `--reload` reinicia la aplicación cuando guardas archivos de Python. Para detener el servidor, presiona **Ctrl+C** en su terminal.

## Prueba los endpoints existentes

Antes de modificar el código, comprueba que la API funciona. Estas rutas ya están implementadas en [`main.py`](main.py):

| Método | Ruta | Resultado |
| --- | --- | --- |
| `GET` | `/` | Redirige a `/docs` con el estado HTTP `301`. |
| `GET` | `/countries` | Devuelve un arreglo JSON con los nombres de los países disponibles. |
| `GET` | `/countries/{country}/{city}/{month}` | Devuelve los valores mensuales `high` y `low` de una ciudad. |

`GET /countries/{country}` **todavía no está implementado**. El ejercicio consiste en añadirlo. La ruta de información meteorológica mensual ya existe y tiene otra función.

### ¿Qué devuelve `/countries/{country}/{city}/{month}`?

Este endpoint responde a la pregunta: **«¿Qué valores históricos de temperatura máxima y mínima están guardados para esta ciudad y este mes?»** Busca una entrada en [`weather.json`](weather.json) y la devuelve como JSON.

Las partes entre llaves son marcadores de posición. Sustitúyelas por valores concretos en la URL, sin escribir las llaves:

| Parámetro de ruta | Significado | Ejemplo |
| --- | --- | --- |
| `{country}` | País al que pertenece la ciudad. | `Spain` |
| `{city}` | Ciudad disponible para ese país en `weather.json`. | `Seville` |
| `{month}` | Nombre del mes en inglés, con las mayúsculas y minúsculas usadas en los datos. | `January` |

Por ejemplo, para consultar **Sevilla, España, en enero**, envía una solicitud `GET` a esta URL o ábrela directamente en el navegador mientras el servidor está en ejecución:

```text
http://127.0.0.1:8000/countries/Spain/Seville/January
```

En Codespaces, sustituye `http://127.0.0.1:8000` por la URL del puerto de tu codespace. No necesitas enviar un cuerpo en la solicitud. La respuesta tiene el estado HTTP **200** y este contenido:

```json
{"high": 61, "low": 41}
```

- `high`: valor de temperatura máxima guardado para Sevilla en enero: **61**.
- `low`: valor de temperatura mínima guardado para Sevilla en enero: **41**.

La aplicación devuelve los valores que ya están en el archivo; no calcula un promedio ni consulta el tiempo actual o un pronóstico. No hay un parámetro de año ni de día: `January` selecciona la entrada de enero del archivo, no una fecha específica. Los datos no indican las unidades de temperatura ni el periodo histórico de referencia.

### Prueba desde el navegador con Swagger UI

FastAPI proporciona un cliente interactivo para la API en `/docs`. No necesitas instalar Postman ni otro cliente HTTP.

1. Abre **http://127.0.0.1:8000/docs** o la URL del puerto de tu codespace seguida de `/docs`.
2. Despliega **GET /countries**, selecciona **Try it out** y luego **Execute**.
3. En **Server response**, comprueba que el estado HTTP sea **200** y que el cuerpo JSON sea:

   ```json
   ["England", "France", "Germany", "Peru", "Portugal", "Italy", "Spain"]
   ```

4. Despliega **GET /countries/{country}/{city}/{month}** y selecciona **Try it out**.
5. Escribe `Spain` en **country**, `Seville` en **city** y `January` en **month**. Después, selecciona **Execute**.
6. Comprueba que el estado HTTP sea **200** y que el cuerpo de la respuesta sea:

   ```json
   {"high": 61, "low": 41}
   ```

Usa los nombres en inglés y respeta las mayúsculas y minúsculas de [`weather.json`](weather.json): `Spain`, `Seville` y `January`, en lugar de `spain`, `Sevilla` o `enero`. La ruta mensual actual no maneja los datos inexistentes; un país, una ciudad o un mes desconocidos provocan un error HTTP `500`.

### Prueba desde una terminal

Ejecuta estas solicitudes en una segunda terminal mientras el servidor esté en ejecución. Envían las mismas solicitudes `GET` que Swagger UI y no necesitan un cuerpo ni autenticación.

**Windows PowerShell:**

```powershell
Invoke-RestMethod -Method Get -Uri "http://127.0.0.1:8000/countries"
Invoke-RestMethod -Method Get -Uri "http://127.0.0.1:8000/countries/Spain/Seville/January"
```

PowerShell muestra la respuesta interpretada como objetos. Para ver el estado HTTP y el JSON sin procesar, usa `curl.exe`:

```powershell
curl.exe -i "http://127.0.0.1:8000/countries/Spain/Seville/January"
```

**macOS / Linux / Codespaces:**

```bash
curl -i "http://127.0.0.1:8000/countries"
curl -i "http://127.0.0.1:8000/countries/Spain/Seville/January"
```

La opción `-i` incluye los encabezados de la respuesta. Debes obtener `HTTP/1.1 200 OK` seguido del JSON mostrado arriba.

## 💪🏽 Ejercicio

La API actual no expone country/{country}, que debe implementarse para listar las ciudades. La ruta debe permitir únicamente solicitudes HTTP GET, con una respuesta JSON que proporcione información sobre las máximas y mínimas históricas para ese país, ciudad y mes.

Como en cualquier implementación, este cambio debe incluir al menos una función de prueba que se ejecute con pytest.

**Aclaración sobre la ruta:** la ruta que debes añadir es `/countries/{country}`, como se muestra en el ejemplo original de abajo. Devuelve las ciudades disponibles para un país. Los valores históricos `high` y `low` ya se obtienen mediante `/countries/{country}/{city}/{month}`; no necesitas sustituir ese endpoint. Para España, la nueva ruta debe responder con el estado HTTP `200` y `["Seville"]`.

### 🛠 Paso 1: Añade una nueva ruta

En el primer ejercicio crearemos una nueva ruta en la API. Abre el archivo main.py y usa el chat en línea con el atajo `ctrl` + `i` en Windows o `cmd` + `i` en Mac. Pide a GitHub Copilot que te ayude a crear una API que muestre las ciudades de un país.

`> Create a new route that exposes the cities of a country.`

Este prompt debería producir algo similar a lo siguiente:

```python
# Create a new route that exposes the cities of a country:
@app.get('/countries/{country}')
def cities(country: str):
    return list(data[country].keys())

```

Nota: prueba la nueva ruta y ajusta el prompt hasta obtener el resultado esperado.

#### Indicaciones adicionales: guarda y prueba la nueva ruta

Usa el atajo de chat en línea y el prompt originales que aparecen arriba. Coloca la ruta generada en `main.py` **antes** de la sección `# Generate the OpenAPI schema:`, que llama a `app.openapi()`. De lo contrario, el endpoint podría aceptar solicitudes sin aparecer en `/docs`. Conserva las rutas y el diccionario `data` existentes.

Guarda `main.py` y espera a que Uvicorn recargue la aplicación. Después, comprueba la ruta con Swagger UI:

1. Abre **http://127.0.0.1:8000/docs** o la URL del puerto de tu codespace seguida de `/docs`.
2. Actualiza la página y despliega **GET /countries/{country}**.
3. Selecciona **Try it out**, escribe `Spain` en **country** y selecciona **Execute**.
4. En **Server response**, comprueba el estado HTTP **200** y el cuerpo JSON `["Seville"]`.

También puedes enviar la misma solicitud desde una segunda terminal mientras el servidor está en ejecución.

**Windows PowerShell:**

```powershell
curl.exe -i "http://127.0.0.1:8000/countries/Spain"
```

**macOS / Linux / Codespaces:**

```bash
curl -i "http://127.0.0.1:8000/countries/Spain"
```

Debes obtener `HTTP/1.1 200 OK` y `["Seville"]`. Antes de implementar la ruta, esta URL devuelve `404` porque todavía no existe. El ejemplo original da por hecho que el país está presente en `weather.json`; para este laboratorio, usa `Spain` con esa mayúscula inicial.

### 🔎 Paso 2: Crear una prueba
Ahora que creaste una ruta, vamos a usar Copilot Chat para crear una prueba de esa ruta con España como país. Recuerda seleccionar tu código y pedirle a Copilot Chat que te ayude con esta API específica que acabamos de crear.

`> /tests help me to create a new test for this route that uses Spain as the country.`

#### Guía adicional: elegir la herramienta de pruebas

La primera vez que uses `/tests`, VS Code podría pedirte que configures las pruebas de Python. Esta configuración permite trabajar con el prompt original de arriba.

1. En **Select a test framework/tool to enable**, elige **pytest — pytest framework**. Si **unittest** está resaltado, haz clic en **pytest** o selecciónalo con las flechas y presiona **Enter**. Este repositorio incluye `pytest` en `requirements.txt` y usa funciones con nombres que comienzan con `test_...` en `test_main.py`.
2. Si te pregunta qué carpeta contiene las pruebas, selecciona **`.` / Root directory**, la carpeta que contiene `test_main.py`. No necesitas crear una carpeta `tests`.
3. Con pytest, normalmente no hace falta indicar un patrón de nombres de archivo. Sus patrones predeterminados, `test_*.py` y `*_test.py`, ya incluyen `test_main.py`. Si seleccionaste `unittest` y te pide un patrón, cancela y vuelve a configurar las pruebas con pytest.
4. Abre la vista **Testing** (el ícono del matraz) y confirma que se detecten las pruebas existentes `test_root` y `test_countries`. Guarda la prueba nueva para que también se detecte.

Si seleccionaste la herramienta equivocada, abre la paleta de comandos (**Ctrl+Shift+P** en Windows/Linux o **Cmd+Shift+P** en Mac), ejecuta **Python: Configure Tests** y elige **pytest** y la carpeta raíz. Si es necesario, revisa la configuración del área de trabajo: `python.testing.pytestEnabled` debe ser `true` y `python.testing.unittestEnabled` debe ser `false`. Consulta [Pruebas de Python en VS Code](https://code.visualstudio.com/docs/python/testing#_configure-tests).

Si no aparece ninguna prueba, ejecuta **Python: Select Interpreter** desde la paleta de comandos y selecciona el entorno del proyecto: `/home/vscode/venv/bin/python` en Codespaces o el intérprete local de `.venv`. Después ejecuta **Test: Refresh Tests**. El intérprete debe tener instaladas las dependencias de `requirements.txt`; consulta la sección de configuración anterior.

Para conocer los patrones predeterminados de nombres de archivo, consulta la [documentación de detección de pruebas de pytest](https://docs.pytest.org/en/stable/how-to/usage.html).

![Ejemplo de Copilot Chat](./images/ideascopilot.png)

**¿Qué significa «pruébala» en el texto de abajo?** Guarda la prueba de Python generada dentro del proyecto y ejecútala con pytest siguiendo [las instrucciones para guardar y ejecutar la prueba que aparecen más abajo](#guía-adicional-guardar-y-ejecutar-la-prueba). Verifica que la solicitud a `/countries/Spain` devuelva HTTP `200` y exactamente `["Seville"]`. Si la prueba pasa, continúa con el paso 3; el prompt de corrección que aparece abajo es un ejemplo para una prueba que falla porque la lista de ciudades esperada es incorrecta.


Cuando Copilot te haya ayudado a crear la prueba, pruébala. Si no funciona como esperabas, comparte esos detalles con Copilot en el chat. Por ejemplo:

`> This test is not quite right, it is not including cities that doesn't exist. Only Seville is part of the API.`


Debería proponerte otra solución. Sigue intentándolo hasta obtener el resultado esperado.


#### Guía adicional: guardar y ejecutar la prueba

##### Agregar la prueba generada al archivo existente

Selecciona la ruta nueva en `main.py`, incluidos su decorador `@app.get(...)` y el cuerpo de la función, y envía el prompt original de `/tests` que aparece arriba. Elige pytest y la carpeta raíz si VS Code te pide configurar las pruebas.

El código que aparece en Copilot Chat es una propuesta; es posible que todavía no se haya agregado a un archivo. Sigue estos pasos para incorporar la prueba al proyecto:

1. En **Explorer** de VS Code, abre el archivo **existente** [`test_main.py`](test_main.py), junto a `main.py`. Ya contiene las importaciones, `client = TestClient(app)`, `test_root` y `test_countries`.
2. Revisa si Copilot ya insertó una prueba nueva en ese archivo. Si lo hizo, revisa los cambios y no vuelvas a copiarla y pegarla; continúa con el guardado del punto 5. Conserva las dos pruebas existentes.
3. Si la prueba solo está en el chat, copia **únicamente la función de prueba nueva**, desde la línea `def test_...` hasta su última aserción. No copies los delimitadores de los bloques de código Markdown ni dupliques las importaciones o la configuración de `client` que ya están en el archivo. En el ejemplo de abajo, la función se llama `test_cities_for_spain`.
4. Ve al final de `test_main.py`, después de la función completa `test_countries`. Deja dos líneas en blanco y pega ahí la función nueva. Su línea `def` debe comenzar al inicio de la línea, alineada con las otras líneas `def`; el cuerpo usa cuatro espacios de sangría. No debe quedar dentro de `test_countries` ni en `main.py`.

Este es el bloque nuevo que debes agregar al final:

```python
def test_cities_for_spain():
    """Spain returns only the city included in the dataset."""
    response = client.get("/countries/Spain")
    assert response.status_code == 200
    assert response.json() == ["Seville"]
```

5. Con la pestaña de edición de `test_main.py` activa, presiona **Ctrl+S** en Windows/Linux o **Cmd+S** en Mac para guardar. Guarda también `main.py` si aún no has guardado los cambios de la ruta. Dejar el código en el chat no guarda una prueba.

El archivo ahora debe contener las importaciones y la configuración de `client` originales, las dos funciones de prueba originales y la función de prueba nueva una sola vez. Si Copilot usó otro nombre para la función, usa ese nombre en el comando de abajo en lugar de `test_cities_for_spain`.

Antes de ejecutarla, compara la URL de la solicitud con la ruta original del paso 1: `/countries/{country}`, por lo que la solicitud para España es `/countries/Spain`. Si Copilot propuso `/countries/{country}/cities`, revisa esa diferencia con respecto al ejemplo original antes de continuar; agregar `/cities` cambia la URL que usa este laboratorio.

Esta prueba realiza o verifica tres acciones concretas:

| Código | Qué hace o verifica |
| --- | --- |
| `client.get("/countries/Spain")` | Envía una solicitud `GET` a la ruta nueva para España usando el cliente de pruebas de la aplicación. |
| `assert response.status_code == 200` | Verifica que la solicitud se haya completado correctamente. |
| `assert response.json() == ["Seville"]` | Verifica que la respuesta sea exactamente la lista de ciudades de `weather.json`, sin ciudades adicionales. |

##### Ejecutar la prueba guardada y verificar el resultado

Abre **Terminal > New Terminal** en VS Code para conservar abierta la terminal del servidor. Ejecuta los siguientes comandos desde la raíz del repositorio, la carpeta que contiene `main.py` y `test_main.py`, con el entorno virtual de la sección de configuración activado.

Primero, ejecuta únicamente la prueba nueva:

```bash
python -m pytest test_main.py::test_cities_for_spain -v
```

En Codespaces puedes llamar directamente al intérprete configurado:

```bash
/home/vscode/venv/bin/python -m pytest test_main.py::test_cities_for_spain -v
```

Debe aparecer una línea que contenga `test_main.py::test_cities_for_spain PASSED` y un resumen que incluya **`1 passed`**. Aquí, `test_main.py::test_cities_for_spain` selecciona una función de un archivo; no ejecuta todo el conjunto de pruebas.

Después verifica que las pruebas existentes sigan pasando:

```bash
python -m pytest -q
```

En Codespaces usa `/home/vscode/venv/bin/python -m pytest -q` si es necesario. El repositorio inicial tiene **2 pruebas**. Con la única prueba nueva de arriba, la ejecución completa debe mostrar **`3 passed`**. Si Copilot agregó más pruebas, el total será mayor.

Ejecuta estos comandos con pytest; usar **Run Python File** en `test_main.py` no ejecuta estas funciones de prueba. `TestClient` ejecuta la aplicación dentro del mismo proceso, por lo que las pruebas no requieren que Uvicorn esté en ejecución ni que haya un puerto reenviado.

##### Si la prueba falla, usar el resultado real para corregirla

Lee la salida de la terminal antes de enviar otro prompt. El error puede estar en la ruta, la prueba generada o el entorno. Usa estas comprobaciones para identificar cuál necesita atención:

| Resultado en la terminal | Qué revisar después |
| --- | --- |
| `not found` para la prueba solicitada | Guarda `test_main.py` y verifica que el nombre de la función coincida con el nombre después de `::` en el comando. |
| Solo `2 passed` al ejecutar todo el conjunto de pruebas | Confirma que la función nueva se haya guardado en `test_main.py`, que su nombre comience con `test_` y que no esté dentro de otra función. |
| La aserción del estado muestra `404` en lugar de `200` | Guarda `main.py` y verifica que la ruta nueva exista en `/countries/{country}` y que la prueba haga la solicitud a `/countries/Spain`. |
| La aserción del JSON muestra listas de ciudades diferentes | Compara la respuesta real y la lista esperada con `weather.json`. España solo contiene `Seville`; una expectativa generada que incluya `Madrid`, por ejemplo, es incorrecta. |
| `KeyError` | Revisa las claves de los datos y las mayúsculas y minúsculas, en especial `Spain`. `TestClient` puede mostrar la excepción de la aplicación directamente en la traza del error. |
| `IndentationError` | Alinea el `def` de la función nueva con las funciones existentes y aplica cuatro espacios de sangría a su cuerpo. |
| `ModuleNotFoundError` para una dependencia | Usa el entorno virtual del proyecto e instala `requirements.txt` como se explica en la configuración. |

Por ejemplo, si la prueba generada espera ciudades adicionales, usa el prompt de corrección original de arriba e incluye la evidencia. Un prompt de seguimiento más detallado puede ser:

```text
I ran: python -m pytest test_main.py::test_cities_for_spain -v
Expected behavior: GET /countries/Spain returns status 200 and ["Seville"].
Actual failure:
[Paste the failing assertion and relevant traceback here.]
Review main.py, test_main.py, and weather.json. Explain whether the route
or the assertion needs correcting, and propose the change.
```

Revisa la corrección propuesta, guarda el archivo modificado y vuelve a ejecutar el comando de la prueba individual. Cuando pase, vuelve a ejecutar todo el conjunto de pruebas. Continúa con el paso 3 cuando pasen tanto la prueba nueva como las existentes.

### 🐍 Paso 3: Usa un agente para escribir la documentación del proyecto
En este paso usaremos un agente (workspace) para escribir la documentación que explica cómo ejecutar el proyecto. En GitHub Copilot Chat, probaremos el siguiente prompt:

`> @workspace help me to use an agent to write the project documentation on how to run it .`


#### Guía adicional: revisa la documentación generada

Usa el prompt original con `@workspace` que aparece arriba. Compara la documentación propuesta con `main.py`, `requirements.txt` y `test_main.py`. Debe explicar desde dónde ejecutar los comandos, cómo instalar las dependencias e iniciar Uvicorn, y cómo enviar una solicitud y comparar su respuesta con el JSON esperado. Las secciones de configuración y verificación de este README incluyen ejemplos concretos que puedes usar como referencia.

Por último, verifica que el nuevo endpoint funcione: abre el endpoint `/docs` y confirma que aparezca en la documentación.

Para esta verificación final, actualiza `/docs`, expande **GET /countries/{country}**, selecciona **Try it out**, escribe `Spain` y selecciona **Execute**. Confirma el código HTTP **200** y la respuesta `["Seville"]`; después ejecuta `python -m pytest -q` para confirmar que pasen las pruebas existentes y la nueva prueba.



🚀 ¡Felicidades! En este ejercicio usaste Copilot para generar código de forma interactiva y divertida. Puedes usar GitHub Copilot para generar código, escribir documentación, probar tus aplicaciones y mucho más.



## Solución de problemas

| Síntoma | Qué revisar o hacer |
| --- | --- |
| No se encuentra `python` en Windows | Prueba `py --version` y después usa `py` para crear el entorno virtual. Una vez activado, usa `python` para los comandos restantes. |
| PowerShell bloquea `Activate.ps1` | Puedes omitir la activación y llamar al entorno directamente: `.\.venv\Scripts\python.exe -m pip install -r requirements.txt` y después `.\.venv\Scripts\python.exe -m uvicorn main:app --host 127.0.0.1 --port 8000 --reload`. Para las pruebas, usa `.\.venv\Scripts\python.exe -m pytest -q`. |
| `No module named fastapi`, `uvicorn` o `pytest` | Activa el entorno virtual y ejecuta `python -m pip install -r requirements.txt`. En Codespaces, usa `/home/vscode/venv/bin/python`. |
| `Could not import module "main"` o `No module named webapp` | Ejecuta el comando desde la carpeta que contiene `main.py` y usa `main:app`. Este repositorio no tiene un paquete `webapp`. |
| El puerto 8000 ya está en uso | Revisa si la API ya está en ejecución. Detén tu servidor anterior con Ctrl+C, o usa `--port 8001` y cambia las URL de los ejemplos al puerto 8001. En Codespaces, configura el reenvío de ese puerto en la pestaña Ports. |
| El navegador o curl no pueden conectarse | Mantén abierta la terminal de Uvicorn y revisa si hay errores en los mensajes de inicio. En Codespaces, abre el puerto 8000 desde la pestaña Ports. |
| `/countries/Spain` devuelve `404` | Completa el Paso 1, guarda `main.py` y espera a que se recargue la aplicación. Esta ruta no existe en el código inicial. |
| La nueva ruta no aparece en `/docs` | Coloca su definición antes de `app.openapi()`, reinicia el servidor si es necesario y actualiza el navegador. Usa `/docs` de la aplicación en ejecución; los archivos de esquema JSON del repositorio pueden estar desactualizados. |
| Una solicitud de datos del clima devuelve `500` | Revisa los nombres exactos del país, la ciudad y el mes en inglés en `weather.json`; después consulta la traza del error del servidor. No está implementado el manejo de datos faltantes para la ruta mensual. |

## Archivos del proyecto

| Archivo | Propósito |
| --- | --- |
| [`main.py`](main.py) | Aplicación FastAPI, rutas y generación del esquema. |
| [`weather.json`](weather.json) | Datos de países, ciudades y clima mensual que usa la API. |
| [`test_main.py`](test_main.py) | Pruebas automatizadas con pytest y el `TestClient` de FastAPI. |
| [`requirements.txt`](requirements.txt) | Dependencias de Python para ejecutar y probar la aplicación. |
| [`.devcontainer/`](.devcontainer/) | Configuración del contenedor de desarrollo para Codespaces y VS Code. |

# Avisos legales

Microsoft y todos los colaboradores te otorgan una licencia sobre la documentación de Microsoft y los demás contenidos
de este repositorio conforme a la [Licencia Pública Internacional de Atribución 4.0 de Creative Commons](https://creativecommons.org/licenses/by/4.0/legalcode),
consulta el archivo [LICENSE](LICENSE), y te otorgan una licencia sobre todo el código del repositorio conforme a la [Licencia MIT](https://opensource.org/licenses/MIT), consulta el
archivo [LICENSE-CODE](LICENSE-CODE).

Microsoft, Windows, Microsoft Azure y otros productos y servicios de Microsoft mencionados en la documentación
pueden ser marcas comerciales o marcas registradas de Microsoft en Estados Unidos y otros países.
Las licencias de este proyecto no te otorgan derechos para usar nombres, logotipos ni marcas comerciales de Microsoft.
Puedes consultar las directrices generales sobre marcas comerciales de Microsoft en http://go.microsoft.com/fwlink/?LinkID=254653.

Puedes consultar la información de privacidad en https://privacy.microsoft.com/en-us/

Microsoft y todos los colaboradores se reservan los demás derechos, ya sea en virtud de sus respectivos derechos de autor, patentes
o marcas comerciales, por implicación, impedimento legal o cualquier otro motivo.
