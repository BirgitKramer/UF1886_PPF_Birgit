# BLOQUE 3 – Monitorización y detección de incidencias
## 1. Objetivo
Durante la ejecución del proceso ETL se ha realizado una monitorización del
sistema con el objetivo de:
- revisar los logs del proceso ETL
- verificar el estado de la base de datos
- analizar el rendimiento del sistema
Además, se han identificado posibles incidencias que pueden afectar al proceso
de transporte de datos.
---
# 2. Monitorización del proceso ETL
## 2.1 Revisión de logs de Apache Hop
Durante la ejecución del pipeline `P02_transport_data.hpl`, Apache Hop genera
registros de ejecución que permiten analizar el estado del proceso.
Los logs muestran información sobre:
- inicio y fin del pipeline
- número de registros procesados
- pasos ejecutados
  - posibles errores
Ejemplo de mensaje de log observado:
Pipeline execution finished successfully
Los logs permiten detectar problemas como:
- errores de conexión a la base de datos
- fallos en transformaciones
- errores durante la inserción de datos
---
# 3. Monitorización de la base de datos
## 3.1 Verificación de conexiones activas
Para analizar el estado de PostgreSQL se pueden revisar las conexiones activas
en la base de datos.
Consulta utilizada:
```sql
SELECT pid, usename, datname, state, query
FROM pg_stat_activity;
```

Esta consulta permite observar:
- número de conexiones activas
- estado de las sesiones
- actividad de consultas
Esto ayuda a detectar posibles problemas como:
- exceso de conexiones
- sesiones bloqueadas
- consultas en ejecución prolongada.