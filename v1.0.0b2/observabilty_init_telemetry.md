# 🔌 Inicialización y Auto-Instrumentación

## Concepto Principal

Antes de iniciar el registro y la recolección de métricas a través de los diversos componentes (como el decorador o el middleware), es fundamental inicializar la conexión con el proveedor de observabilidad.

La función `init_telemetry` establece el vínculo entre la aplicación y **Azure Monitor** o un **Gateway OTLP**, encargándose de inyectar correctamente la cadena de conexión o endpoint y la telemetría base de OpenTelemetry. Esta configuración **solo debe ejecutarse una vez** al inicio de la aplicación.

---

## Parámetros de `init_telemetry` y Variables de Entorno

La función `init_telemetry` acepta varios parámetros opcionales. Sin embargo, puede ser configurada íntegramente mediante variables de entorno, lo cual es la práctica recomendada.

### Destinos de Telemetría (Se requiere al menos uno)
No se puede inicializar la telemetría sin enviar los datos a un sink. Tienes que definir al menos uno de los siguientes:
- **`otlp_endpoint`** (`str`, Opcional): URL del servidor Gateway OTLP.
  - *Variable de entorno*: `OTEL_EXPORTER_OTLP_ENDPOINT`
- **`appinsights_connection_string`** (`str`, Opcional): Cadena de conexión directa hacia Application Insights de Azure Monitor.
  - *Variable de entorno*: `APPLICATIONINSIGHTS_CONNECTION_STRING`

*(NOTA: Si no configuras ninguno de los destinos, la función arrojará un error `RuntimeError ("No telemetry destination configured...")`).*

### Configuraciones Generales
- **`service_name`** (`str`, Opcional): Nombre del servicio para asociar a los registros de rastreo.
  - *Variable de entorno*: `OTEL_SERVICE_NAME`
- **`capture_message_content`** (`bool`, Opcional): Flag para indicar si se deben capturar los _prompts_ y respuestas de los de LLMs internos. Útil en depuración.
- **`trace_to_console`** (`bool`, Opcional): Flag para emitir todos los spans localmente por terminal, únicamente recomendado para el desarrollo o modo interactivo.

---

## Patrón Singleton para la Inicialización

Para garantizar que la inicialización ocurra estrictamente una única vez, se recomienda implementar un patrón **Singleton**. Este centraliza el estado de la conexión en un módulo específico y previene instancias múltiples del trazador.

### Implementación (`telemetry_singleton.py`)

A continuación, se presentan dos ejemplos de la implementación recomendada según el destino configurado para tu telemetría. En ambos modelos destaca la inclusión de `enable_auto_instrumentation()`, función encargada de activar la **Auto-instrumentación**. La auto-instrumentación permite capturar eventos generados internamente por frameworks soportados (ej. Microsoft Agent Framework) sin tener que instanciar trazados de forma manual.

### Opción A: Inicialización para Gateway OTLP

Utiliza esta configuración si envías los datos hacia un servidor OTLP (asegúrate de instalar la librería como `monitoria[observability]`).

```python
from azure.core.settings import settings

settings.tracing_implementation = "opentelemetry"

import os
import logging
from dotenv import load_dotenv

from monitoria.observability import init_telemetry, enable_auto_instrumentation

load_dotenv()

log = logging.getLogger(__name__)

tracer = None
trace_link = None
otel_sink_server = None
_initialized = False


def setup_init_telemetry():
    global tracer, trace_link, otel_sink_server, _initialized

    if _initialized:
        return tracer, trace_link, otel_sink_server

    os.environ.setdefault("OTEL_SERVICE_NAME", os.getenv("OTEL_SERVICE_NAME", "poc-agentes-credicorp"))
    os.environ.setdefault("ENABLE_INSTRUMENTATION", "true")
    os.environ.setdefault("ENABLE_SENSITIVE_DATA", os.getenv("ENABLE_SENSITIVE_DATA", "false"))

    otlp_endpoint = os.getenv("OTEL_EXPORTER_OTLP_ENDPOINT", "https://wapceu2aiasd00.eastus2-01.azurewebsites.net")

    tracer, trace_link, otel_sink_server = init_telemetry(
        otlp_endpoint=otlp_endpoint,
        service_name=os.environ["OTEL_SERVICE_NAME"],
        capture_message_content=os.getenv("OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT", "false").lower() in (
        "1", "true", "yes"),
        trace_to_console=os.getenv("OTEL_TRACE_TO_CONSOLE", "false").lower() in ("1", "true", "yes")
    )

    enable_auto_instrumentation()

    _initialized = True
    return tracer, trace_link, otel_sink_server
```

### Opción B: Inicialización para Azure Monitor (Application Insights)

Utiliza esta configuración si fuerzas el envío a Application Insights configurando explícitamente tú la cadena de conexión (asegúrate de usar la librería instalada como `monitoria[observability-az-monitor]`).

```python
import os
import logging
from dotenv import load_dotenv

from monitoria.observability import init_telemetry, enable_auto_instrumentation

load_dotenv()

log = logging.getLogger(__name__)

tracer = None
trace_link = None
otel_sink_server = None
_initialized = False

AZURE_CONNECTION_STRING = "InstrumentationKey=45c8513c-b045-4478-8167-..."

if "OTEL_EXPORTER_OTLP_ENDPOINT" in os.environ:
    del os.environ["OTEL_EXPORTER_OTLP_ENDPOINT"]


def setup_init_telemetry():
    global tracer, trace_link, otel_sink_server, _initialized

    if _initialized:
        return tracer, trace_link, otel_sink_server

    os.environ.setdefault("OTEL_SERVICE_NAME", os.getenv("OTEL_SERVICE_NAME", "poc-agentes-credicorp"))
    os.environ.setdefault("ENABLE_INSTRUMENTATION", "true")
    os.environ.setdefault("ENABLE_SENSITIVE_DATA", os.getenv("ENABLE_SENSITIVE_DATA", "false"))

    print("Inicializando telemetría hacia Azure Monitor (Application Insights)...")

    # Inicializamos la telemetría usando la cadena de conexión de App Insights.
    tracer, trace_link, otel_sink_server = init_telemetry(
        appinsights_connection_string=AZURE_CONNECTION_STRING,
        service_name=os.environ["OTEL_SERVICE_NAME"],
        capture_message_content=os.getenv("OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT", "false").lower() in (
        "1", "true", "yes"),
        trace_to_console=os.getenv("OTEL_TRACE_TO_CONSOLE", "false").lower() in ("1", "true", "yes")
    )

    enable_auto_instrumentation()

    _initialized = True
    return tracer, trace_link, otel_sink_server
```

---

### Integración en la Aplicación

Una vez configurado, cualquier otro módulo en la aplicación puede requerir las variables de telemetría llamando al Singleton:

```python
from app import telemetry_singleton as telemetry

# Inicializa o devuelve la instancia si ya fue activado
telemetry.setup_init_telemetry()

# Reutilizar las variables globales del trazador
tracer = telemetry.tracer
otel_sink_server = telemetry.otel_sink_server
```

## 📝 Notas sobre la configuración
- Abstenerse de ejecutar `init_telemetry` de forma independiente a lo largo del código. Utilice la función `setup_init_telemetry` del Singleton.
- Es crucial asegurarse de que las credenciales de Azure Monitor o el endpoint OTLP (`OTEL_EXPORTER_OTLP_ENDPOINT`) según corresponda estén configuradas en las variables de entorno.
- El identificador `trace_link` servirá como contexto base de todo el ciclo de ejecución de la instancia.