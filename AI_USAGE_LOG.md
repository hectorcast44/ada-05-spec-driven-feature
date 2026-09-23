# AI Usage Log — ADA-05: Customer Search Feature

Este registro documenta las interacciones clave de Inteligencia Artificial que influyeron en las fases de requisitos, especificación, arquitectura, descomposición de tareas, implementación, pruebas y revisión durante el desarrollo del proyecto **ADA-05 (Spec-Driven Feature)**.

---

## Entrada 1: Definición de Requisitos y Especificación Formal
- **Fecha / Momento**: Fase 1 y Fase 2 (Requirements & Spec)
- **Etapa del Flujo**: `REQUIREMENTS.md` y `SPEC.md`
- **Propuesta de la IA**:
  - La IA sugirió una lista de requisitos funcionales que incluía búsqueda con operadores booleanos complejos (`AND`, `OR`, `NOT`) y soporte para expresiones regulares (regex) en la interfaz CLI.
  - Sugirió además redefinir el texto completo de cada requisito dentro de `SPEC.md`.
- **Decisión del Estudiante / Desarrollador**:
  - **Rechazar** el soporte de regex y operadores booleanos complejos para mantener el alcance enfocado en la búsqueda parcial (*substring match*) e insensibilidad a mayúsculas/minúsculas requerida en la consigna de ADA-05.
  - **Ajustar** la regla clave de spec-driven development: `SPEC.md` no debe redefinir ni duplicar los requisitos, sino referenciarlos por sus identificadores estables (`FR-01` a `FR-06`, `NFR-01` a `NFR-03`) y transformarlos en reglas verificables (`SR-01` a `SR-05`, `VR-01` a `VR-03`) y criterios de aceptación (`AC-01` a `AC-09`).
  - Definir la regla de validación de longitud mínima de 2 caracteres (`VR-02`) para evitar consultas ambiguas o barridos innecesarios.
- **Impacto en el Producto**:
  - Se formalizó un contrato de especificación limpio, sin redundancias y perfectamente verificable, estableciendo las bases para pruebas automatizadas directas.

---

## Entrada 2: Diseño de Arquitectura y Decisiones de Persistencia
- **Fecha / Momento**: Fase 3 (Architecture)
- **Etapa del Flujo**: `ARCHITECTURE.md`
- **Propuesta de la IA**:
  - Implementar una base de datos SQLite con SQLAlchemy o Pydantic como ORM/validador de esquemas, y exponer un servidor HTTP FastAPI.
- **Decisión del Estudiante / Desarrollador**:
  - **Rechazar** dependencias pesadas como SQLAlchemy o FastAPI para la solución núcleo, priorizando la restricción de herramientas gratuitas, simplicidad y portabilidad (Python 3.11+ estándar con dataclasses nativas y CLI con `argparse`).
  - **Aceptar** el diseño en capas desacopladas (*Domain*, *Repository*, *Service*, *Validator*, *CLI*) con abstracción `CustomerRepository` mediante `Protocol`.
  - Diseñar el repositorio en memoria con persistencia opcional basada en archivo JSON (`data/customers.json`).
- **Impacto en el Producto**:
  - Cero dependencias externas en tiempo de ejecución. Tiempos de ejecución ultra rápidos (< 10 ms), alta mantenibilidad y facilidad para mockear/testear en `pytest`.

---

## Entrada 3: Descomposición de Tareas y Control de Baseline en Git
- **Fecha / Momento**: Fase 4 a Fase 6 (Tasks & Git Baseline)
- **Etapa del Flujo**: `TASKS.md`, `AGENTS.md` y commit inicial
- **Propuesta de la IA**:
  - Proceder directamente a escribir el código de los modelos y del repositorio antes de formalizar el commit baseline en Git.
- **Decisión del Estudiante / Desarrollador**:
  - **Pausar** la implementación de código inmediatamente.
  - Respetar de forma estricta la regla metodológica de ADA-05: generar el commit inicial de Git (`spec: define customer search feature`) conteniendo únicamente los artefactos de especificación (`REQUIREMENTS.md`, `SPEC.md`, `ARCHITECTURE.md`, `TASKS.md`, `AGENTS.md`, `.gitignore`) antes de tocar una sola línea de código fuente.
- **Impacto en el Producto**:
  - Trazabilidad histórica impecable en el repositorio Git, garantizando que el diseño y la especificación precedieron formalmente a la implementación.

---

## Entrada 4: Detección y Corrección en Pruebas de Búsqueda por Email
- **Fecha / Momento**: Fase 7 (Implementation & Testing - T-03)
- **Etapa del Flujo**: `tests/test_service.py` y `CustomerSearchService`
- **Propuesta de la IA**:
  - En la prueba `test_search_by_email_partial_and_case_insensitive`, la IA escribió una aserción esperando `assert result.total_matches == 1` para la consulta `"garcia"` en el campo email, asumiendo que solo `cust-003` (`bmendez@garcia-associates.com`) coincidía.
  - Al ejecutar `pytest`, la prueba falló con `assert 2 == 1`.
- **Decisión del Estudiante / Desarrollador**:
  - Analizar el dataset de clientes y la regla de búsqueda parcial. Se identificó que `cust-001` (`ana.garcia@example.com`) también contenía `"garcia"` en su dirección de correo electrónico.
  - En lugar de forzar o debilitar la lógica del servicio para ignorar el email de `cust-001`, se corrigió la aserción del test para verificar que ambos clientes (`cust-001` y `cust-003`) son correctamente retornados, y se añadió una verificación adicional para el dominio específico `"garcia-associates.com"`.
- **Impacto en el Producto**:
  - Preservación íntegra de la regla de coincidencia parcial case-insensitive (`SR-02`, `SR-03`, `FR-02`), evitando bugs sutiles en la validación de dominios y nombres de usuario de correo.

---

## Entrada 5: Experiencia de Usuario en la CLI y Control de Errores
- **Fecha / Momento**: Fase 7 (Implementation - T-05)
- **Etapa del Flujo**: `src/customer_search/cli.py` y `tests/test_cli.py`
- **Propuesta de la IA**:
  - Lanzar las excepciones directamente al sistema permitiendo que Python muestre el traceback estándar si ocurría un error de validación o si no existía el archivo JSON.
- **Decisión del Estudiante / Desarrollador**:
  - **Rechazar** el comportamiento por defecto de Python. De acuerdo con `FR-06`, `AC-06` y `AGENTS.md`, la CLI nunca debe exponer tracebacks al usuario final ante errores de entrada o datos no encontrados.
  - Implementar captura de `ValidationError` y `DataLoadError` emitiendo un mensaje formateado a `sys.stderr` (`Error: <mensaje>`) y retornando código de salida `1`.
  - Asegurar código de salida `0` cuando no hay coincidencias pero la consulta es válida, informando amigablemente al usuario.
- **Impacto en el Producto**:
  - Interfaz de línea de comandos robusta, amigable y apta para scripts bash o automatizaciones POSIX.
