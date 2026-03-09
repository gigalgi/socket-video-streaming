# 📹 VideoStreaming — Node.js + Python + OpenCV

**Real-time video streaming bridge between Python (OpenCV) and the browser, using Socket.IO as the transport layer.**
**Puente de transmisión de video en tiempo real entre Python (OpenCV) y el navegador, usando Socket.IO como capa de transporte.**

[![License](https://img.shields.io/badge/License-MIT-lightgrey)](LICENSE)
![Node](https://img.shields.io/badge/Node.js-Express%20%7C%20Socket.IO-green)
![Python](https://img.shields.io/badge/Python-OpenCV%20%7C%20python--socketio-blue)

---

## 🇬🇧 English

### How it works

The system has three components that can be mixed and matched:

```
┌──────────────────┐        WebSocket (Socket.IO)        ┌──────────────────┐
│  EMITTER         │ ─────── base64 JPEG frames ────────► │  RECEIVER        │
│  emitir.py       │                                      │  visualizar.py   │
│  emitir.html     │        ┌─────────────────┐           │  visualizar.html │
└──────────────────┘        │  Node.js Server │           └──────────────────┘
                            │  app.js         │
                            │  Socket.IO relay│
                            └─────────────────┘
```

The Node.js server acts as a relay — it receives a `stream` event from any connected emitter and immediately broadcasts it to all other connected clients. The emitter and receiver are interchangeable between Python and browser:

| Emit from | Receive in | Use case |
|---|---|---|
| `emitir.py` (OpenCV) | `visualizar.html` (browser) | Monitor a camera feed in the browser |
| `emitir.html` (browser webcam) | `visualizar.py` (OpenCV) | Capture browser camera → post-process in Python |
| `emitir.py` | `visualizar.py` | Python-to-Python relay over a network |
| `emitir.html` | `visualizar.html` | Browser-to-browser relay |

Each frame is encoded as a JPEG at 50% quality, base64-encoded, and sent as a string over Socket.IO. This keeps bandwidth low while maintaining acceptable latency for real-time use.

### Prerequisites

**Node.js server:**
```bash
npm i
```

**Python scripts:**
```bash
pip install opencv-python python-socketio numpy
```

### Running

**1. Start the Node.js relay server:**
```bash
npm run dev       # development mode (auto-reload via nodemon)
# or
npm start         # production mode
```
Server listens on `http://localhost:3000` by default.

**2. Choose your emitter and receiver:**

```bash
# Option A — Emit from Python, view in browser
python emitir.py
# Then open http://localhost:3000/visualizar.html

# Option B — Emit from browser, receive in Python for post-processing
# Open http://localhost:3000/emitir.html in your browser
python visualizar.py

# Option C — Browser to browser
# Open http://localhost:3000/emitir.html      (sender tab)
# Open http://localhost:3000/visualizar.html  (receiver tab)
```

Press `Esc` to stop the Python scripts.

### Project Structure

```
├── app.js                  # Express + Socket.IO relay server
├── package.json
├── emitir.py               # Python emitter — captures webcam, streams to server
├── visualizar.py           # Python receiver — receives stream, displays with OpenCV
└── public/
    ├── index.html          # Landing page
    ├── emitir.html         # Browser emitter — captures webcam via getUserMedia
    └── visualizar.html     # Browser receiver — displays incoming stream
```

### Extending for computer vision pipelines

`visualizar.py` receives each frame as a decoded NumPy array, making it straightforward to insert any OpenCV or ML post-processing step before display:

```python
# Inside decode_img() in visualizar.py — insert your processing here:
imgn = cv2.imdecode(nparr, cv2.IMREAD_COLOR)

# Examples:
imgn = cv2.cvtColor(imgn, cv2.COLOR_BGR2GRAY)   # Grayscale
imgn = cv2.Canny(imgn, 100, 200)                # Edge detection
# results = model.detect(imgn)                  # Object detection
# imgn = apply_ar_overlay(imgn, results)        # AR overlay
```

The processed frame can be re-emitted back to the browser through the same Socket.IO channel, creating a full browser → Python → browser post-processing loop.

### Configuration

| Parameter | Location | Default | Notes |
|---|---|---|---|
| Server port | `app.js` | `3000` | Override via `PORT` env variable |
| Camera resolution | `emitir.py` | `640 × 420` | `cap.set(3, width)` / `cap.set(4, height)` |
| JPEG quality | `emitir.py` / `emitir.html` | `50%` | Lower = smaller frames |
| Frame rate | `emitir.html` | ~33 fps | `setInterval(..., 30)` — interval in ms |
| Frame delay | `emitir.py` | none | Uncomment `sio.sleep(x)` to throttle |

---

## 🇨🇴 Español

### Cómo funciona

El sistema tiene tres componentes que se pueden combinar libremente:

```
┌──────────────────┐        WebSocket (Socket.IO)        ┌──────────────────┐
│  EMISOR          │ ─────── frames JPEG en base64 ─────► │  RECEPTOR        │
│  emitir.py       │                                      │  visualizar.py   │
│  emitir.html     │        ┌─────────────────┐           │  visualizar.html │
└──────────────────┘        │  Servidor       │           └──────────────────┘
                            │  Node.js        │
                            │  app.js         │
                            └─────────────────┘
```

El servidor Node.js actúa como relé — recibe el evento `stream` de cualquier emisor conectado y lo retransmite inmediatamente a todos los demás clientes. El emisor y el receptor son intercambiables entre Python y el navegador:

| Emitir desde | Recibir en | Caso de uso |
|---|---|---|
| `emitir.py` (OpenCV) | `visualizar.html` (navegador) | Monitorear la cámara desde el navegador |
| `emitir.html` (webcam del navegador) | `visualizar.py` (OpenCV) | Capturar desde el navegador → procesar en Python |
| `emitir.py` | `visualizar.py` | Relé Python a Python por red |
| `emitir.html` | `visualizar.html` | Relé navegador a navegador |

Cada fotograma se codifica como JPEG al 50% de calidad, se convierte a base64 y se envía como string por Socket.IO. Esto mantiene el ancho de banda bajo con latencia aceptable.

### Requisitos previos

**Servidor Node.js:**
```bash
npm i
```

**Scripts Python:**
```bash
pip install opencv-python python-socketio numpy
```

### Ejecución

**1. Inicia el servidor relé Node.js:**
```bash
npm run dev       # modo desarrollo (recarga automática con nodemon)
# o
npm start         # modo producción
```
El servidor escucha en `http://localhost:3000` por defecto.

**2. Elige tu emisor y receptor:**

```bash
# Opción A — Emitir desde Python, ver en el navegador
python emitir.py
# Luego abre http://localhost:3000/visualizar.html

# Opción B — Emitir desde el navegador, recibir en Python para post-procesado
# Abre http://localhost:3000/emitir.html en el navegador
python visualizar.py

# Opción C — Navegador a navegador
# Abre http://localhost:3000/emitir.html      (pestaña emisora)
# Abre http://localhost:3000/visualizar.html  (pestaña receptora)
```

Presiona `Esc` para detener los scripts de Python.

### Estructura del proyecto

```
├── app.js                  # Servidor Express + Socket.IO
├── package.json
├── emitir.py               # Emisor Python — captura webcam y transmite al servidor
├── visualizar.py           # Receptor Python — recibe el stream y muestra con OpenCV
└── public/
    ├── index.html          # Página de inicio
    ├── emitir.html         # Emisor en el navegador — captura webcam via getUserMedia
    └── visualizar.html     # Receptor en el navegador — muestra el stream recibido
```

### Extensión para pipelines de visión por computador

`visualizar.py` recibe cada fotograma como un array NumPy decodificado, lo que permite insertar cualquier procesado OpenCV o ML antes de mostrarlo:

```python
# Dentro de decode_img() en visualizar.py — inserta tu procesado aquí:
imgn = cv2.imdecode(nparr, cv2.IMREAD_COLOR)

# Ejemplos:
imgn = cv2.cvtColor(imgn, cv2.COLOR_BGR2GRAY)   # Escala de grises
imgn = cv2.Canny(imgn, 100, 200)                # Detección de bordes
# results = model.detect(imgn)                  # Detección de objetos
# imgn = apply_ar_overlay(imgn, results)        # Overlay de realidad aumentada
```

El fotograma procesado puede re-emitirse de vuelta al navegador por el mismo canal Socket.IO, creando un bucle completo navegador → Python → navegador.

### Configuración

| Parámetro | Ubicación | Valor por defecto | Notas |
|---|---|---|---|
| Puerto del servidor | `app.js` | `3000` | Sobreescribir con variable de entorno `PORT` |
| Resolución de cámara | `emitir.py` | `640 × 420` | `cap.set(3, ancho)` / `cap.set(4, alto)` |
| Calidad JPEG | `emitir.py` / `emitir.html` | `50%` | Menor = fotogramas más livianos |
| Tasa de fotogramas | `emitir.html` | ~33 fps | `setInterval(..., 30)` — intervalo en ms |
| Retardo entre frames | `emitir.py` | ninguno | Descomentar `sio.sleep(x)` para limitar la tasa |

---

## Autor / Author

**Gilberto Galvis Giraldo**

---

## Licencia / License

MIT — see [LICENSE](LICENSE) for details.
