# ADA-05: Spec-Driven Feature — Customer Search
**Ingeniería de Software asistida por IA · UADY**

Implementación de una funcionalidad de **Customer Search** en Python siguiendo la metodología **Spec-Driven Development (SDD)**:
`REQUIREMENTS.md` → `SPEC.md` → `ARCHITECTURE.md` → `TASKS.md` → `AGENTS.md` → `Git Baseline` → `Implementation` → `Tests` → `Traceability` → `AI Usage Log`.

---

## 1. Estructura del Repositorio

La disposición del proyecto cumple con la estructura de entrega estipulada en la sección 16 del documento ADA-05:

```
ada-05-spec-driven-feature/
├── README.md                      # Documentación principal, ejecución y reflexión
├── REQUIREMENTS.md                # Fuente de verdad de los requisitos (FR/NFR)
├── SPEC.md                        # Especificación formal y criterios de aceptación
├── ARCHITECTURE.md                # Diseño de componentes, flujo y diagramas
├── TASKS.md                       # Descomposición en tareas atómicas T-01 a T-06
├── AGENTS.md                      # Reglas de operación para el coding agent
├── AI_USAGE_LOG.md                # Registro de interacciones significativas con IA
├── data/
│   └── customers.json             # Dataset de clientes de prueba en JSON
├── src/
│   └── customer_search/
│       ├── __init__.py
│       ├── models.py              # Entidades Customer y SearchResult
│       ├── exceptions.py          # Jerarquía de excepciones personalizadas
│       ├── validator.py           # Validador de consultas (VR-01, VR-02, VR-03)
│       ├── repository.py          # Abstracción y repositorio en memoria / JSON
│       ├── service.py             # Lógica central de búsqueda y filtrado
│       └── cli.py                 # Interfaz de línea de comandos (CLI)
├── tests/
│   ├── __init__.py
│   ├── conftest.py                # Fixtures compartidos para pytest
│   ├── test_models.py             # Pruebas unitarias de modelos de dominio
│   ├── test_validator.py          # Pruebas unitarias de validación
│   ├── test_repository.py         # Pruebas de acceso a datos y carga JSON
│   ├── test_service.py            # Pruebas de integración del servicio de búsqueda
│   ├── test_cli.py                # Pruebas de integración de la interfaz CLI
│   └── test_performance.py        # Prueba de rendimiento (< 100 ms para 1000 registros)
├── docs/
│   └── traceability.md            # Matriz de trazabilidad Requirement -> Test
└── results/
    └── agent-report.md            # Informe de ejecución del agente por tareas
```

---

## 2. Requisitos Previos e Instalación

### Requisitos
- Python 3.11 o superior (probado en Python 3.13.12).
- Git.
- Entorno virtual (recomendado).

### Instalación
```bash
# 1. Clonar el repositorio y navegar a la raíz
cd ada-05-sdd

# 2. Crear y activar el entorno virtual
python3 -m venv .venv
source .venv/bin/activate

# 3. Instalar la única dependencia de desarrollo (pytest)
pip install pytest
```

---

## 3. Guía de Uso del CLI

El comando principal se ejecuta como módulo de Python: `python -m src.customer_search.cli`.

### Opciones Disponibles
```
usage: customer-search [-h] [-f {all,name,email}] [-d DATA_FILE] [--json] query

Search customers by name or email with partial match support.

positional arguments:
  query                 Search term (minimum 2 non-whitespace characters).

options:
  -h, --help            show this help message and exit
  -f, --field {all,name,email}
                        Field to search within (default: all).
  -d, --data-file DATA_FILE
                        Path to JSON customer data file.
  --json                Output results in JSON format.
```

### Ejemplos de Ejecución

#### 1. Búsqueda unificada por término (por defecto en nombre y email)
```bash
python -m src.customer_search.cli "garcia"
```
*Salida:*
```
Found 3 customer(s) matching 'garcia' [field: all]:
--------------------------------------------------------------------------------
ID           | NAME                         | EMAIL                          | STATUS  
--------------------------------------------------------------------------------
cust-001     | Ana Sofia Garcia             | ana.garcia@innovatech.mx       | Active  
cust-003     | Beatriz Elena Mendez         | bmendez@garcia-associates.com  | Inactive
cust-002     | Carlos Alberto Ramirez Garcia | carlos.ramirez@globex.org      | Active  
--------------------------------------------------------------------------------
```

#### 2. Búsqueda filtrada específicamente por correo electrónico
```bash
python -m src.customer_search.cli "innovatech.mx" --field email
```

#### 3. Búsqueda filtrada específicamente por nombre
```bash
python -m src.customer_search.cli "torres" --field name
```

#### 4. Salida en formato JSON estructurado
```bash
python -m src.customer_search.cli "torres" --json
```
*Salida:*
```json
{
  "query": "torres",
  "field": "all",
  "total_matches": 1,
  "customers": [
    {
      "id": "cust-005",
      "name": "Elena Maria Torres",
      "email": "elena.torres@acmewidgets.com",
      "phone": "+52-999-567-8901",
      "is_active": true
    }
  ]
}
```

#### 5. Manejo controlado de errores de validación (código de salida 1 sin tracebacks)
```bash
python -m src.customer_search.cli "a"
# Salida a stderr: Error: Search query must contain at least 2 non-whitespace characters.
```

---

## 4. Ejecución de Pruebas Automatizadas

La suite comprende 42 pruebas automatizadas que cubren unit tests, integration tests, CLI tests y performance tests:

```bash
# Ejecutar toda la suite con salida detallada
pytest -v

# Ejecutar una suite específica
pytest tests/test_service.py -v
pytest tests/test_cli.py -v
pytest tests/test_performance.py -v
```

---

## 5. Respuestas a las Preguntas de Reflexión (Sección 18)

### 1. ¿Qué diferencia hay entre requirement, specification y prompt?
- **Requirement (`REQUIREMENTS.md`)**: Es la declaración del **qué** necesita el usuario o negocio desde una perspectiva de producto (e.g., *"El usuario necesita buscar clientes por nombre o correo"*). Es independiente de la tecnología, estable y tiene identificadores fijos (`FR-01`).
- **Specification (`SPEC.md`)**: Es el contrato técnico verificable que traduce los requerimientos a reglas operativas, modelos de dominio, validaciones, pre/post-condiciones y criterios de aceptación comprobables (e.g., *"La consulta debe tener $\ge 2$ caracteres tras recorte, compararse con `lower()` en subcadena y ordenarse alfabéticamente"*).
- **Prompt**: Es la instrucción operativa contextual que se envía a un modelo de IA en un momento determinado para que ejecute una tarea específica (e.g., *"Implementa la clase `CustomerSearchService` siguiendo las reglas `SR-01` a `SR-05` de `SPEC.md`"*). El prompt es efímero; el requirement y la specification son artefactos persistentes del proyecto.

### 2. ¿Qué información fue indispensable antes de programar?
Fue indispensable contar con:
- La definición de los campos de la entidad `Customer` (`id`, `name`, `email`, `phone`, `is_active`).
- Las reglas de normalización y búsqueda (insensibilidad a mayúsculas, coincidencia de subcadenas, deduplicación en búsqueda unificada).
- Las reglas de validación de entrada (mínimo 2 caracteres significativos, rechazo de cadenas en blanco).
- El comportamiento ante errores y casos límite (sin resultados = lista vacía y código 0; consulta inválida = mensaje limpio a `stderr` y código 1 sin traceback).

### 3. ¿Qué decisiones debieron resolverse antes de programar?
- **Persistencia**: Se decidió utilizar un repositorio desacoplado en memoria con capacidad de carga desde archivo JSON local, evitando dependencias de bases de datos pesadas innecesarias para el alcance.
- **Tratamiento de espacios en blanco**: Resolver si `"  ana  "` se tomaba literal o requería recorte (`strip`). Se resolvió que el recorte es automático previo a la validación.
- **Longitud mínima de búsqueda**: Fijar el límite en 2 caracteres (`VR-02`) para prevenir búsquedas sobre una sola letra que devolverían colecciones excesivas.
- **Criterio de ordenamiento**: Definir orden alfabético por nombre y desempate por identificador.

### 4. ¿Qué errores evitó SPEC.md?
- Evitó que el agente implementara búsqueda exacta en lugar de búsqueda parcial (*substring*).
- Evitó duplicados de clientes en búsquedas combinadas cuando un cliente tiene el término tanto en su nombre como en su correo.
- Evitó que la CLI reventara con un `IndexError`, `ValueError` o traceback no controlado cuando el usuario pasa argumentos inválidos o archivos inexistentes.
- Evitó desacuerdos entre lo que las pruebas verificaban y lo que el código implementaba.

### 5. ¿Qué problema evita separar REQUIREMENTS.md de SPEC.md?
Evita el acoplamiento entre los objetivos de negocio y las decisiones de diseño técnico. Cuando se mezclan, cualquier cambio de implementación corre el riesgo de alterar silenciosamente el requerimiento de negocio, o viceversa: un cambio de requerimiento desestabiliza toda la arquitectura. Al separarlos, `REQUIREMENTS.md` actúa como la fuente de verdad inmutable del producto y `SPEC.md` actúa como el contrato de ingeniería verificable que referencia pero no redefine dichos requerimientos.

### 6. ¿Qué papel tuvo AGENTS.md?
Funcionó como la "constitución" de operación del coding agent. Estableció límites explícitos para:
- Prohibir inventar requisitos no solicitados.
- Prohibir alterar o debilitar pruebas automatizadas cuando algo fallaba.
- Forzar la lectura de especificaciones antes de codificar.
- Mantener cambios pequeños, atómicos y respaldados por `pytest`.

### 7. ¿Qué cambió durante la revisión humana?
- **Corrección en pruebas de búsqueda por email**: El agente inicialmente asumió en un test que el término `"garcia"` solo pertenecía a un cliente (`bmendez@garcia-associates.com`), ignorando que `cust-001` tenía por correo `ana.garcia@example.com`. La revisión humana detectó que el fallo del test no era culpa del servicio, sino de una aserción errónea en el test que no consideraba el dataset completo.
- **Control de baseline en Git**: Se aseguró que el commit inicial contuviera exclusivamente la especificación antes de iniciar la implementación.
- **Manejo de códigos de salida en la CLI**: Se verificó que los errores de validación no generasen tracebacks en la consola.

### 8. ¿La arquitectura coincidió con el código final?
Sí, en un 100%. Los componentes planificados en `ARCHITECTURE.md` (`Domain Models`, `Repository Protocol`, `InMemoryCustomerRepository`, `SearchQueryValidator`, `CustomerSearchService`, `CLI`) se implementaron exactamente con las responsabilidades, signaturas e interfaces definidas, sin desviaciones.

### 9. ¿Qué requisito fue más difícil de probar?
El requisito **FR-06** combinado con **FR-04** en la interfaz de usuario (CLI): asegurar que ante cualquier entrada inválida (como strings vacíos, caracteres únicos o archivos inexistentes) la aplicación emitiera el mensaje preciso a `stderr`, saliera con código de salida `1` y garantizara la ausencia total de cualquier línea de traceback de Python, capturando `SystemExit` y redirigiendo streams mediante fixtures de pytest (`capsys`).

### 10. ¿Qué mejorarías en tu proceso spec-driven?
- Automatizar la verificación de trazabilidad mediante una herramienta de linting de especificaciones que valide programáticamente que cada identificador en `REQUIREMENTS.md` posea al menos un criterio de aceptación en `SPEC.md` y una función `test_*` en el directorio `tests/`.
- Incorporar pruebas basadas en propiedades (*property-based testing* con Hypothesis) para probar combinaciones aleatorias de caracteres especiales y acentos en el validador y normalizador de consultas.

### 11. ¿Por qué SPEC.md puede funcionar como contrato verificable sin duplicar los requisitos?
Porque utiliza una relación de **mapeo por referencia**. Al listar `Requirements Covered: [FR-01, FR-02, ...]`, `SPEC.md` asume la existencia de los requerimientos y se enfoca exclusivamente en definir las reglas matemáticas y de comportamiento verificables (`SR-xx`, `VR-xx`) y los criterios de aceptación medibles (`AC-xx`). De esta forma, si el texto descriptivo del requisito cambia en `REQUIREMENTS.md`, la especificación técnica no se desincroniza ni sufre de inconsistencias textuales.

### 12. ¿Qué decisiones importantes aparecen en tu AI Usage Log y cómo cambió tu criterio después de revisar las sugerencias de la IA?
Aparecen decisiones cruciales como:
1. **Rechazo a la sobreingeniería**: La IA sugirió añadir bases de datos externas (SQLite/SQLAlchemy) o APIs HTTP complejas (FastAPI). El criterio humano mantuvo la arquitectura en el stack mínimo necesario (Python estándar + `pytest`), logrando un sistema más rápido, confiable y libre de dependencias externas.
2. **Defensa de la integridad de los requisitos**: Ante el fallo del test de email, la tentación de un agente suele ser "parchar" el código para que el test pase o eliminar la coincidencia en el servicio; el análisis humano identificó que la regla de búsqueda parcial estaba operando a la perfección y que era la prueba la que debía ajustarse al dataset real.
3. **Respeto a las fases metodológicas**: Evitar escribir código antes del commit de Git baseline de especificaciones, consolidando la disciplina de ingeniería guiada por especificación.
