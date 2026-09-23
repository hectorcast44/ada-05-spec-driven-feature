# Agent Report

## Agent / Version
- **Agent Name**: Antigravity (Google DeepMind - Advanced Agentic Coding)
- **Model**: Gemini 3.8 Flash (High)
- **Environment**: macOS / Python 3.13.12 / pytest 9.1.1 / Git 2.50.1

## Initial Context
El proyecto **ADA-05 (Spec-Driven Feature)** requirió construir desde cero una funcionalidad de **Customer Search** que soporte búsqueda parcial, insensibilidad a mayúsculas/minúsculas, validación de consultas y pruebas automatizadas, siguiendo un riguroso ciclo de ingeniería de software especificado en 10 fases (Requirements → Specification → Architecture → Tasks → Agents → Git baseline → Implementation → Traceability → Tests → Human Review).
El repositorio inicial se encontraba completamente vacío sin código ni configuración previa.

## Task Sequence

### T-01 Project Setup
- **What the agent did**:
  - Creó la estructura de directorios (`src/customer_search/`, `tests/`, `data/`, `docs/`, `results/`).
  - Creó el entorno virtual `.venv` e instaló `pytest`.
  - Configuró `.gitignore` para omitir artefactos generados, caches y entornos virtuales.
  - Creó `tests/conftest.py` con fixtures compartidos para pruebas.
- **Human review**:
  - Se validó que el entorno virtual no fuera rastreado por Git y que `pytest` estuviera accesible en el entorno.
- **Tests**:
  - `pytest` se inicializó y comprobó recolección sin errores.

### T-02 Domain Model
- **What the agent did**:
  - Implementó la clase inmutable `Customer` y el contenedor `SearchResult` en `src/customer_search/models.py` utilizando `dataclasses` nativas con validación de tipos y métodos `to_dict()` / `from_dict()`.
  - Creó la suite de pruebas unitarias `tests/test_models.py`.
- **Human review**:
  - Se confirmó que los modelos cumplen exactamente con el esquema de datos estipulado en `SPEC.md`.
- **Tests**:
  - `tests/test_models.py` (7 pruebas unitarias pasaron satisfactoriamente en 0.01s).

### T-03 Search Logic
- **What the agent did**:
  - Definió el protocolo `CustomerRepository` y la implementación `InMemoryCustomerRepository` con capacidades de carga y deserialización desde archivos JSON en `src/customer_search/repository.py`.
  - Creó el dataset de prueba representativo en `data/customers.json`.
  - Implementó el servicio `CustomerSearchService` en `src/customer_search/service.py` con coincidencia parcial (`substring`), insensibilidad a mayúsculas (`SR-02`), búsqueda combinada en todos los campos (`SR-04`) y ordenamiento alfabético (`SR-05`).
  - Creó pruebas unitarias e integración en `tests/test_repository.py` y `tests/test_service.py`.
- **Human review**:
  - Se detectó una aserción errónea en la prueba de búsqueda por correo donde no se consideró que el término `"garcia"` estaba presente en dos registros (`cust-001` y `cust-003`). Se ordenó corregir la prueba para validar la regla real sin alterar la lógica correcta del servicio.
- **Tests**:
  - `tests/test_repository.py` (6 pruebas) y `tests/test_service.py` (7 pruebas) pasaron al 100%.

### T-04 Validation and Errors
- **What the agent did**:
  - Definió la jerarquía de excepciones en `src/customer_search/exceptions.py` (`CustomerSearchError`, `ValidationError`, `DataLoadError`).
  - Implementó el validador `SearchQueryValidator` en `src/customer_search/validator.py` aplicando `VR-01` (no vacío), `VR-02` (mínimo 2 caracteres significativos tras trim) y `VR-03` (campos válidos).
  - Creó la suite de pruebas `tests/test_validator.py`.
- **Human review**:
  - Se verificó que los mensajes de excepción sean consistentes con lo redactado en `SPEC.md`.
- **Tests**:
  - `tests/test_validator.py` (12 casos parametrizados pasaron en 0.01s).

### T-05 CLI & Performance Tests
- **What the agent did**:
  - Implementó la interfaz de línea de comandos en `src/customer_search/cli.py` con soporte para `--field`, `--json`, `--data-file` y formateo de texto tabular.
  - Implementó el manejo de excepciones para emitir errores a `stderr` con código de salida `1` sin imprimir tracebacks.
  - Creó la suite de pruebas de integración para la CLI en `tests/test_cli.py` y pruebas de rendimiento en `tests/test_performance.py` (1,000 clientes sintéticos en < 100 ms).
- **Human review**:
  - Se comprobó manualmente la ejecución de la CLI en la terminal con casos válidos, casos vacíos y flags JSON.
- **Tests**:
  - `tests/test_cli.py` (9 pruebas) y `tests/test_performance.py` (1 prueba) pasaron exitosamente.
  - Total de la suite: **42 pruebas pasando en 0.03s**.

### T-06 Documentation
- **What the agent did**:
  - Elaboró la matriz de trazabilidad en `docs/traceability.md`.
  - Creó el registro estructurado `AI_USAGE_LOG.md` con 5 entradas representativas.
  - Redactó este reporte de agente en `results/agent-report.md`.
  - Elaboró el archivo `README.md` con la guía completa de instalación, ejecución y las respuestas a las 12 preguntas de reflexión.
- **Human review**:
  - Verificación cruzada de identificadores (`FR-xx`, `NFR-xx`, `AC-xx`, `T-xx`) y coherencia documental.
- **Tests**:
  - Validación de enlaces y verificación de ejecución limpia del pipeline completo.

## Problems Encountered
1. **Fallo en aserción de búsqueda por correo**:
   - *Problema*: Al probar la búsqueda parcial en el campo de email con el término `"garcia"`, el test asumía 1 coincidencia (`bmendez@garcia-associates.com`), pero en realidad existían 2 coincidencias (`ana.garcia@example.com` y `bmendez@garcia-associates.com`).
   - *Solución*: Se actualizó la prueba para validar el comportamiento real estipulado en la especificación (ambos correos contienen `"garcia"` como subcadena).
2. **Ambiente de ejecución en Sandbox restringido**:
   - *Problema*: El sandbox inicial impedía la descarga de dependencias externas por red durante la instalación de paquetes con pip.
   - *Solución*: Se solicitó autorización explícita para instalar `pytest` mediante bypass sandbox y se mantuvo la ejecución de pruebas y desarrollo dentro de sandbox estándar sin requerir accesos de red posteriores.

## Human Interventions
1. **Aprobación del Implementation Plan**: Revisión y aprobación del plan de 10 fases antes de iniciar la creación de archivos.
2. **Control de Git Baseline**: Verificación de que el commit inicial contuviera exclusivamente los artefactos de especificación antes de escribir la implementación.
3. **Revisión de mensajes de error de la CLI**: Confirmación de que ningún error de validación exponga tracebacks de Python en terminal.

## Requirement / Specification Changes
No existieron cambios ni adiciones a los requerimientos originales de negocio (`REQUIREMENTS.md` y `SPEC.md`) durante la implementación. Las preguntas abiertas iniciales (`Q-01`, `Q-02`) fueron resueltas antes de programar y se mantuvieron estables en todo el ciclo de desarrollo.

## Final Verification
- **Pruebas Automatizadas**: 42 pruebas ejecutadas mediante `pytest -v` con 100% de éxito en 0.03 segundos.
- **Pruebas de Rendimiento**: Consulta sobre 1,000 registros sintéticos ejecutada en ~4.5 ms (ampliamente por debajo del umbral de 100 ms).
- **Pruebas Manuales de CLI**: Verificadas exitosamente para salida tabular, salida JSON, filtrado por campo y control de errores.

## Lessons Learned
1. **El valor de una especificación desacoplada**: Separar `REQUIREMENTS.md` de `SPEC.md` evita la ambigüedad y previene que el equipo altere los requisitos para adaptarse a limitaciones accidentales de la implementación.
2. **Pruebas como especificación viva**: Cada criterio de aceptación (`AC-xx`) se convirtió en un test de pytest directamente auditable mediante la matriz de trazabilidad.
3. **El rol de AGENTS.md**: Las restricciones explícitas en `AGENTS.md` evitaron que el asistente agregara dependencias pesadas innecesarias o modificara requerimientos sin consulta previa.
