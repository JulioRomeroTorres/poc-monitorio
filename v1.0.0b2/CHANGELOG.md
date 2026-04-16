# Changelog
Todas las novedades de esta librería se documentan en este fichero.

## [1.0.0b2] - 2026-04-14
  - refactor: Reubicación de la metadata universal (`session_id`, `otel_sink_server` y `additional_properties`) directamente al nivel de atributos de Recurso (Resource Attributes) desde `init_telemetry()`.
  - feature: El atributo `otel_sink_server`  ahora se resuelve internamente para capturar el otel sink (appinsight / otel collector).
  - breaking-change: Eliminación de los parámetros redundantes `session_id` y `server_host` de las funciones `GenAITrackingSSEMiddleware`, `tracking_genai` y `set_genai_span_attrs`.
  - refactor: Renombrados los parámetros internos `service_type` y `operation_type` por `service_name` y `operation_name`.
  - devops: Actualización de `collector.yaml`, mapeo de `resource.attributes` y agrupamiento de trazas (`groupbytrace`).

## [1.0.0b1] - 2026-04-09
  - refactor: Eliminación de la dependencia `azure.ai.projects` del módulo de observabilidad.
  - feature: Reestructuración de dependencias opcionales en pyproject.toml usando `monitoria[observability]` y `monitoria[observability-az-monitor]`.
  - feature: Simplificación de la función `init_telemetry` de `monitoria.observability` eliminando uso del cliente de proyecto, `project_endpoint`, y objetos `credential`. Ahora la inicialización requiere estrictamente `otlp_endpoint` / `OTEL_EXPORTER_OTLP_ENDPOINT` y/o `appinsights_connection_string` / `APPLICATIONINSIGHTS_CONNECTION_STRING`.
  - docs: Actualizada la documentación para documentar los parámetros permitidos e instrucciones de instalación de dependencias extra modificadas.
  - refactor: Se renombró el módulo `tracing` por `observability`.


## [0.2.1] - 2026-04-07
  - feature: Estandarización del tracking del decorador (usando las llaves fijas `custom_function.name`, `custom_function.input`, y `custom_function.output`) consolidando los argumentos como JSON de forma nativa.
  - feature: Extracción de `GenAITrackingSSEMiddleware` hacia la librería base (`monitoria.tracing.middleware`) para estandarizar el registro HTTP.
  - feature: El middleware captura de forma implícita parámetros HTTP asíncronos y almacena el cuerpo completo re-ensamblado en `custom_endpoint.output` soportando tanto iteradores JSON estándar como Event Streams (SSE).
  - feature: Introducción del método global `enable_auto_instrumentation` dentro de `monitoria.tracing` encapsulando la logica de inicializacion del Microsoft Agent Framework.
  - devops: Actualización de los procesadores YAML del collector (ver CHANGELOG de Docker) y documentación de los esquemas generados.

## [0.2.0] - 2026-03-30
  - feature: Añadido soporte para usar gateways OTLP estándar además de Azure Monitor para la trazabilidad.
  - feature: Separación de las dependencias. La instalación de `monitoria` trae los exportadores base OTLP, mientras que `monitoria[azure-monitor]` instala los conectores y librerías opcionales para usar Application Insights.
  - docs: Actualizados los ficheros markdown en `/docs` reflejando las nuevas arquitecturas y métodos de configuración unificados mediante `telemetry_singleton.py`.

## [0.1.1] - 2026-02-19
  - feature: Versión inicial estable con soporte de OpenTelemetry, Auto-instrumentación en Agent Framework y Decoradores.

## [0.0.1] - 2025-12-08
  - feature: Versión inicial de librería - PoC