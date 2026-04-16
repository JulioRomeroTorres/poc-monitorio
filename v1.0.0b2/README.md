# coaa-ai-corp-sdk-utils
Repositorio de código fuente para los SDKs utilitarios de Inteligencia Artificial.

## **Módulos**

- `monitoria.observability` [[Código](src/monitoria/observability/)] [[Documentación](./docs/module_observability.md)]
  *Soporta decoradores, middlewares (FastAPI) y auto-instrumentación (Microsoft Agent Framework) enviados vía OTLP o Azure Monitor.*
- `monitoria.evaluation` [[Código](./src/monitoria/evaluation/)] [[Documentación](./docs/module_evaluation.md)]
  *Motor central de validación métrica y evaluadores estandarizados.*

## **OTEL Collector Docker**
- **OpenTelemetry Collector & Routing** [[Docker README](./docker/README.md)] [[Diccionario de Datos](./docker/docs/collector_schema.md)]
  *Pipeline de YAMLs pre-configurados para enrutar el tráfico capturado nativamente hacia Application Insights y Blob Storage (incluyendo definiciones DDL).*