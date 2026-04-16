# 🛡️ Trazabilidad con Middleware

## ¿Qué es el Middleware de Trazabilidad?

Un **Middleware** es un componente que se sitúa entre la recepción de una petición HTTP y su controlador final en una aplicación web (por ejemplo, en FastAPI). 

Cada vez que un cliente realiza una consulta, el middleware registra automáticamente:
- Marca de tiempo de la petición.
- Método y ruta de acceso bajo `custom_endpoint.name`.
- Los inputs (headers filtrados, query params, y body JSON) unificados bajo un único JSON dentro de `custom_endpoint.input`.
- El status de finalización (`status_code`) y el cuerpo íntegro de la respuesta (`body`) almacenados dentro de `custom_endpoint.output`. En el caso de respuestas asíncronas de tipo *StreamingResponse*, el middleware se encarga de interceptar y ensamblar los flujos de bits en vivo para registrar su carga final como texto normal.

El uso de este middleware, ahora provisto nativamente por `monitoria`, asegura la trazabilidad centralizada de operaciones web y soporta flujos complejos como StreamingResponse (SSE) automáticamente.

---

## 🚀 Implementación en la API

La librería `monitoria` ya encapsula toda la lógica bajo la clase `GenAITrackingSSEMiddleware`. Solo requieres agregarlo en tu aplicación principal (`main.py`).

### Registrar el Middleware en `main.py`

En el archivo principal (`main.py`), añade el Middleware configurando las rutas objetivo:

```python
import uvicorn
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

# ✅ 1. Importar el Middleware nativo de monitoria
from monitoria.observability.middleware import GenAITrackingSSEMiddleware
from app import telemetry_singleton as telemetry


def create_app():
    app = FastAPI()

    # ✅ 2. Inicializar telemetría
    telemetry.setup_init_telemetry()
    tracer = telemetry.tracer
    otel_sink_server = telemetry.otel_sink_server

    # ✅ 3. Mapear rutas e identificar las operaciones
    OPERATION_MAP = {
        "POST:/api/chat": "agent-route-post",
        "POST:/api/chat/stream": "agent-route-post-stream",
        "POST:/api/sessions": "create-session-post",
    }
    ENABLED_PATHS = {"/api/chat", "/api/chat/stream", "/api/sessions"}

    # ✅ 4. Agregar el middleware a la App
    app.add_middleware(
        GenAITrackingSSEMiddleware,
        tracer=tracer,
        service_name="credicorp-dispatcher-middleware",
        operation_map=OPERATION_MAP,
        capture_inputs=True,
        enabled_paths=ENABLED_PATHS,
        carrier_mode="fallback"
    )

    # ... Configuración de rutas (routers) ...

    return app


if __name__ == "__main__":
    app = create_app()
    uvicorn.run(app, host="0.0.0.0", port=8080)
```

Con esta configuración, toda solicitud a las rutas provistas será monitoreada de forma automática, estructurando limpiamente los argumentos y respuestas bajo las keys requeridas por la plataforma para el estándar de métricas.