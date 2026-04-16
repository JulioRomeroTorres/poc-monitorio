# 📚 Módulo de Observabilidad (Observability)

Este módulo es la herramienta principal para monitorear y registrar la actividad dentro de tu aplicación de Inteligencia Artificial. 

El módulo permite capturar métricas importantes como:
- Cantidad de tokens consumidos.
- Latencia y tiempos de respuesta del modelo.
- Excepciones y errores ocurridos.
- Historial de la conversación.

Toda esta información se puede enviar a **Azure Monitor** o a un **Gateway OTLP** utilizando el estándar **OpenTelemetry**.

---

## 📦 Instalación de la librería

Dependiendo de si necesitas enviar la telemetría a un **Gateway OTLP** (`otlp_endpoint`) o directamente a **Azure Monitor** (`appinsights_connection_string`), la instalación varía:

- **Para usar con un Gateway OTLP (solo `otlp_endpoint`):**
  Necesitas el paquete base con soporte de observabilidad (`monitoria[observability]`). Ejemplo en `pyproject.toml`:
  ```toml
  [project]
  name = "orchestrator-agent-backend"
  version = "1.0.0"
  description = "Multi-agent orchestrator system using Microsoft Agent Framework"
  readme = "README.md"
  requires-python = ">=3.10"
  dependencies = [
      "monitoria[observability] @ file:///app/monitoria_tool/monitoria-0.1.1-py3-none-any.whl"
  ]
  ```

- **Para usar con Azure Monitor (`appinsights_connection_string`):**
  Si es necesario enviar a Azure Monitor, instala la variante con dependencias extra de Azure (`monitoria[observability-az-monitor]`). Ejemplo en `pyproject.toml`:
  ```toml
  [project]
  name = "orchestrator-agent-backend"
  version = "1.0.0"
  description = "Multi-agent orchestrator system using Microsoft Agent Framework"
  readme = "README.md"
  requires-python = ">=3.10"
  dependencies = [
      "monitoria[observability-az-monitor] @ file:///app/monitoria_tool/monitoria-0.1.1-py3-none-any.whl"
  ]
  ```

---

## 🚀 Las 3 Formas de Usar Este Módulo

Dependiendo de la arquitectura de tu aplicación y de tus necesidades, existen **3 maneras principales** de implementar la trazabilidad:

### 1. Usando el Decorador (`@tracking_genai`)
Esta es la forma manual y más detallada. Al aplicar el decorador sobre funciones específicas, obtienes control total sobre qué métricas exactas registrar en los componentes y agentes.
👉 [Guía de uso del Decorador](observability_decorator_tracking.md)

### 2. Basado en un Middleware (ej. FastAPI)
Si desarrollas una API web (como FastAPI), puedes usar un Middleware. Este componente intercepta y registra automáticamente todas las peticiones (requests) y respuestas HTTP, sin necesidad de modificar el código interno de cada endpoint.
👉 [Guía de uso del Middleware](observability_middleware.md)

### 3. Con Auto-instrumentación
Si usas frameworks soportados (como Microsoft Agent Framework), puedes activar la **Auto-instrumentación**. Esto permite que el framework registre componentes, métricas y spans automáticamente, reduciendo el código manual necesario.
👉 [Guía de Inicialización y Auto-instrumentación](observability_init_telemetry.md)

---

## 🛠️ Primer Paso Obligatorio: Inicialización

Independientemente del método que elijas, **siempre** debes inicializar la configuración de trazabilidad una única vez al arrancar la aplicación.
👉 [Consulta la guía de Inicialización para configurar la conexión](observability_init_telemetry.md).
