# Documentación del Workflow checklist.yml

Este documento describe las validaciones implementadas en el workflow `checklist.yml` para garantizar que los repositorios cumplan con los estándares establecidos para servicios ESB/ACE.

## Tabla de Contenidos
- [Descripción General](#descripción-general)
- [Activadores del Workflow](#activadores-del-workflow)
- [Jobs y Validaciones](#jobs-y-validaciones)
  - [1. Validar Nombre de Rama](#1-validar-nombre-de-rama)
  - [2. Validar Existencia README.md](#2-validar-existencia-readmemd)
  - [3. Validar Plantilla README](#3-validar-plantilla-readme)
  - [4. Validar Carpetas BD](#4-validar-carpetas-bd)
  - [5. Validar Grupos de Ejecución](#5-validar-grupos-de-ejecución)
  - [6. Validar Rutas y Revisores](#6-validar-rutas-y-revisores)
- [Reglas Específicas por Sección](#reglas-específicas-por-sección)
- [Ejemplos de Validación](#ejemplos-de-validación)

## Descripción General

El workflow `checklist.yml` se ejecuta automáticamente en Pull Requests para validar que la documentación y estructura del repositorio cumplan con los estándares definidos. Implementa múltiples validaciones críticas incluyendo formato del README, configuración de DataPower, y políticas de despliegue.

## Activadores del Workflow

El workflow se activa en los siguientes eventos de Pull Request:
- **Ramas destino**: `main`, `develop`, `quality`, `feature/**`
- **Tipos de evento**: `opened`, `synchronize`, `reopened`, `edited`

## Jobs y Validaciones

### 1. Validar Nombre de Rama

**Propósito**: Asegurar que las ramas sigan la convención de nomenclatura establecida.

**Reglas**:
- El nombre de la rama debe comenzar con uno de los siguientes prefijos:
  - `feature/` - Para nuevas funcionalidades
  - `bugfix/` - Para corrección de errores
  - `hotfix/` - Para correcciones urgentes en producción
  - `release/` - Para preparación de releases
- Después del prefijo, debe contener caracteres alfanuméricos, puntos, guiones bajos o guiones: `[A-Za-z0-9._-]+`

**Ejemplos válidos**:
- `feature/nueva-funcionalidad`
- `bugfix/correccion-validacion`
- `hotfix/error-critico`
- `release/v1.2.3`

**Ejemplos inválidos**:
- `nueva-funcionalidad` (falta prefijo)
- `feature/` (falta nombre después del prefijo)
- `feat/nueva-funcionalidad` (prefijo incorrecto)

### 2. Validar Existencia README.md

**Propósito**: Verificar que existe el archivo `README.md` en la raíz del repositorio.

**Reglas**:
- Debe existir un archivo llamado `README.md` (case-sensitive) en la raíz del proyecto

**Resultado**:
- ❌ Error si el archivo no existe
- ✅ Success si el archivo existe

### 3. Validar Plantilla README

Este es el job más completo que valida la estructura y contenido del README.md según la plantilla estándar.

#### 3.1. Título Principal del Servicio

**Reglas**:
- Debe comenzar con `# ESB_`
- **IMPORTANTE**: Debe contener texto descriptivo después de `ESB_`
- No puede ser solo `ESB_` o `ESB_` seguido únicamente de guiones o guiones bajos
- Debe terminar con un punto (.)

**Formato esperado**: `# ESB_ACE12_NombreServicio.` o `# ESB_NombreServicio.`

**Ejemplos válidos**:
```markdown
# ESB_ACE12_UtilizacionCreditoRotativoPlus.
# ESB_ConsultaSaldos.
# ESB_ProcesoPagos.
```

**Ejemplos inválidos**:
```markdown
# ESB_.
# ESB___.
# ESB_---.
# Servicio de Consulta
```

#### 3.2. Sección INFORMACIÓN DEL SERVICIO

**Reglas**:
- Debe existir el encabezado `## INFORMACIÓN DEL SERVICIO`
- **IMPORTANTE**: Debe contener información descriptiva del servicio **antes** de cualquier subsección (como `### Último despliegue`)
- La descripción no puede estar vacía

**Estructura esperada**:
```markdown
## INFORMACIÓN DEL SERVICIO

[Aquí debe ir la descripción del servicio con al menos un párrafo explicando su funcionalidad]

### Último despliegue
...
```

**Ejemplos válidos**:
```markdown
## INFORMACIÓN DEL SERVICIO

El presente documento expone de manera detallada la funcionalidad y componentes del flujo...

### Último despliegue
```

**Ejemplos inválidos**:
```markdown
## INFORMACIÓN DEL SERVICIO

### Último despliegue
```

#### 3.3. Subsección Último Despliegue

**Reglas**:
- Debe existir la subsección `### Último despliege` (note: con la ortografía actual del template)
- Debe contener una tabla con el encabezado: `|CQ |JIRA | Fecha|`
- La tabla debe tener al menos una fila de datos después del separador `|---|---|---|`
- Todas las celdas deben tener valores (puede ser 'NA' si no aplica)
- No puede haber celdas vacías

**Formato esperado**:
```markdown
### Último despliege

|CQ |JIRA | Fecha|
|---|---|---|
| NA | NA |6/10/2023 (WS) |
```

#### 3.4. Sección Procedimiento de Despliegue

**Reglas**:
- Debe existir el encabezado `## Procedimiento de despliegue`
- Debe contener instrucciones de despliegue (no puede estar vacía)
- Case-insensitive para el título

**Ejemplo válido**:
```markdown
## Procedimiento de despliegue

Aplicar UtilizacionCreditoRotativoPlus.properties a UtilizacionCreditoRotativoPlus.bar y desplegar en los grupos de ejecución:
BOGESERVICIOSWS05_SRV01 BOGESERVICIOSWS05_SRV02
```

#### 3.5. Sección ACCESO AL SERVICIO

##### 3.5.1. DataPower (Interno/Externo)

**Reglas principales**:

1. **Encabezado obligatorio**: 
   - Debe existir al menos una de las subsecciones: `### DataPower Interno :` o `### DataPower Externo :`
   - Si ninguna está presente, se lanza un **warning** (no error) recomendando indicar explícitamente con 'NA'

2. **Contenido no vacío**:
   - Si el encabezado existe, **no puede estar vacío**
   - Como mínimo debe contener una de las siguientes formas si no aplica:
     - "NA"
     - "N/A"
     - "No Aplica"
   - Si contiene una tabla, **debe tener al menos una fila de datos** (no solo el encabezado y separador)
   - Se reporta **error** si la tabla está vacía o no tiene filas de datos

3. **Validación de URLs**:
   - **CRÍTICO**: Las URLs de DataPower deben comenzar con `https://boc201` 
   - **NO se permite**: URLs que comiencen con `https://boc200`
   - Debe cumplir con el formato específico por ambiente (ver tabla abajo)

**Ejemplos válidos de DataPower**:

```markdown
### DataPower Interno :
|AMBIENTE|TIPO COMPONENTE|NOMBRE WSP O MPG|DATAPOWER|ENDPOINT|
|---|---|---|---|---|
|DESARROLLO|WSP|WSServicioInterno|BODPDEV|https://boc201.des.app.bancodeoccidente.net:4806/Servicio/Port|
|CALIDAD|WSP|WSServicioInterno|BODPQAS|https://boc201.testint.app.bancodeoccidente.net:4806/Servicio/Port|
|PRODUCCION|WSP|WSServicioInterno|BODPPRD|https://boc201.prdint.app.bancodeoccidente.net:4806/Servicio/Port|
```

```markdown
### DataPower Interno :
NA
```

```markdown
### DataPower Interno :
N/A
```

```markdown
### DataPower Interno :
No Aplica
```

```markdown
### DataPower Externo :
NA

### DataPower Interno :
NA
```

**Ejemplos inválidos de DataPower**:

```markdown
### DataPower Interno :
|AMBIENTE|TIPO COMPONENTE|NOMBRE WSP O MPG|DATAPOWER|ENDPOINT|
|---|---|---|---|---|


```
❌ Error: Tabla con encabezado pero sin filas de datos

```markdown
### DataPower Interno :
|AMBIENTE|TIPO COMPONENTE|NOMBRE WSP O MPG|DATAPOWER|ENDPOINT|
|---|---|---|---|---|
|DESARROLLO|WSP|WSServicio|BODPDEV|https://boc200.des.app.bancodeoccidente.net:4806/Servicio/Port|
```
❌ Error: URL con boc200 (debe ser boc201)

```markdown
## ACCESO AL SERVICIO

### Endpoint BUS
...
```
⚠️ Warning: No se encontró ninguna subsección DataPower

##### 3.5.2. Validaciones de Formato de URLs DataPower

**Por Ambiente y Tipo**:

| Ambiente | DataPower | Endpoint Interno | Endpoint Externo |
|----------|-----------|------------------|------------------|
| DESARROLLO | BODP*DEV | https://boc201.des.app.bancodeoccidente.net | N/A para externo en DEV |
| CALIDAD | BODP*QAS | https://boc201.testint.app.bancodeoccidente.net | https://boc201.testdmz.app.bancodeoccidente.net |
| PRODUCCION | BODP*PRD | https://boc201.prdint.app.bancodeoccidente.net | https://boc201.prddmz.app.bancodeoccidente.net |

**Validaciones adicionales**:
- El nombre del DataPower debe comenzar con `BODP` y terminar con `DEV`, `QAS` o `PRD` según el ambiente
- Deben existir filas para los 3 ambientes (DESARROLLO, CALIDAD, PRODUCCION)
- Si una tabla tiene todos los valores en NA, la otra tabla también debe tener todos en NA

##### 3.5.3. Endpoint BUS

**Reglas**:
- Debe existir la subsección `### Endpoint BUS`
- Debe contener una tabla con endpoints para los 3 ambientes
- **No puede contener valores NA** (a diferencia de DataPower)
- Los endpoints deben cumplir con el formato por ambiente

**Formatos esperados por ambiente**:

| Ambiente | Formato URL |
|----------|-------------|
| DESARROLLO | https://adbog162e.bancodeoccidente.net:XXXX/... |
| CALIDAD | https://a[dt]bog16[34][de].bancodeoccidente.net:XXXX/... |
| PRODUCCION | https://adbog16[56][ab].bancodeoccidente.net:XXXX/... o https://boc060ap.prd.app.bancodeoccidente.net:XXXX/... |

#### 3.6. Sección CANALES - APLICACIONES

**Reglas**:
- Debe existir el encabezado `## CANALES - APLICACIONES`
- Debe contener contenido (no puede estar vacía)
- Debe tener una fila `**Consumidor**` con valores
- Debe tener una fila `**Backends**` con valores
- Ambas filas pueden contener 'NA' si no aplica, pero no pueden estar vacías

**Formato esperado**:
```markdown
## CANALES - APLICACIONES

|||||
|---|---|---|---|
|**Consumidor**|CANALES|PB|IVR|

|||||
|---|---|---|---|
|**Backends**|NA|||
```

#### 3.7. Sección DEPENDENCIAS

**Reglas**:
- Debe existir el encabezado `## DEPENDENCIAS`
- Debe contener dos tablas: `Servicios` y `XSL`

##### 3.7.1. Tabla Servicios

**Reglas**:
- Debe contener la lista de servicios (proyectos) que son dependencias
- Los servicios listados deben coincidir con los proyectos declarados en el archivo `.project`
- Se valida sincronización entre README y archivo `.project`

**Validaciones**:
- ✅ Servicios en `.project` deben estar en README
- ✅ Servicios en README deben existir en `.project`
- ❌ Error si hay discrepancias

##### 3.7.2. Tabla XSL

**Reglas**:
- Debe existir la tabla `|XSL|`
- Si no hay XSLs a consumir, debe indicarse explícitamente con 'NA'
- No puede estar vacía

**Ejemplos**:
```markdown
|XSL|
|---|
|REQ_ACOperacion_017.xsl|
|REQ_ACOperacion_277.xsl|
```

o

```markdown
|XSL|
|---|
|NA|
```

#### 3.8. Sección DOCUMENTACION

**Reglas**:
- Debe existir el encabezado `## DOCUMENTACION`
- Debe contener los siguientes campos con enlaces válidos:

##### 3.8.1. Campo "Documento de diseño detallado"

**Formato**: `**Documento de diseño detallado:**`
- Debe tener un enlace que comience con `https://bancoccidente.sharepoint.com/:f:/r/sites/BibliotecaAplicaciones/`

##### 3.8.2. Campo "Mapeo"

**Formato**: `**Mapeo:**`
- Debe tener un enlace que comience con `https://bancoccidente.sharepoint.com/:f:/r/sites/BibliotecaAplicaciones/`

##### 3.8.3. Campo "Evidencias"

**Formato**: `**Evidencias (Unitarias/Auditoria/Monitoreo):**`
- Debe tener un enlace que comience con `https://bancoccidente.sharepoint.com/:f:/r/sites/BibliotecaAplicaciones/`

##### 3.8.4. Campo "WSDL"

**Formato**: `**WSDL:**`
- Debe contener la ruta del WSDL en formato: `git\{NOMBRE_REPO}\Broker\WSDL\wsdl\{archivo}.wsdl`
- El `{NOMBRE_REPO}` debe coincidir con el nombre del repositorio extraído del título
- Alternativamente, puede contener solo 'NA' o 'N/A' si no aplica

**Ejemplo**:
```markdown
**WSDL:** <br>
git\ESB_ACE12_UtilizacionCreditoRotativoPlus\Broker\WSDL\wsdl\UtililzacionCreditoRotativoPlus.wsdl
```

#### 3.9. Sección SQL

**Reglas**:
- Debe existir el encabezado `## SQL`
- Debe contener queries SQL para auditoría y monitoreo
- No puede estar vacía
- Los códigos de operación en las queries deben ser **solo números** (sin letras)
- **No se permiten códigos de operación vacíos** (`num_id_tipo_operacion = ''`)

**Validaciones de códigos de operación**:
- En formato `where ... = '123'`: el valor debe contener solo dígitos y no puede estar vacío
- En formato `where ... in('123','456')`: todos los valores deben ser solo dígitos
- ❌ Error si se encuentran letras en los códigos de operación
- ❌ Error si se encuentra un código de operación vacío (`= ''`)

**Nota sobre subsecciones FILTRAR**: Las subsecciones que comienzan con "Filtrar" (como "Filtrar por Codigo de Operacion") **son validadas** y deben cumplir con las mismas reglas.

**Ejemplo válido**:
```sql
select * from admesb.esb_log_auditoria where num_id_tipo_operacion = '99042'
```

**Ejemplos inválidos**:
```sql
select * from admesb.esb_log_auditoria where num_id_tipo_operacion = ''
```
❌ Error: código de operación vacío

```sql
select * from admesb.esb_log_auditoria where num_id_tipo_operacion = '99a9042'
```
❌ Error: código contiene letra 'a'

### 4. Validar Carpetas BD

**Propósito**: Evitar la inclusión de carpetas con información sensible de base de datos.

**Reglas**:
- No debe existir ninguna carpeta llamada "BD" (case-insensitive) en el repositorio
- Se excluye la carpeta `.git` de la búsqueda

**Resultado**:
- ❌ Error si se encuentra alguna carpeta BD
- ✅ Success si no se encuentran carpetas BD

### 5. Validar Grupos de Ejecución

**Propósito**: Verificar que los grupos de ejecución documentados coincidan con la configuración central.

**Reglas**:
1. Extrae el nombre del servicio del título del README (formato `ESB_ACE12_{Servicio}`)
2. Busca en el README la línea "desplegar en los grupos de ejecución:"
3. Extrae los grupos listados (pueden estar en la misma línea o en la siguiente)
4. Descarga el archivo `esb-ace12-general-integration-servers.properties` del repositorio central
5. Compara los grupos del README con los del archivo de configuración

**Validaciones**:
- ✅ Los grupos deben coincidir exactamente (sin importar orden ni uso de comas)
- ❌ Error si hay grupos en README que no están en config
- ❌ Error si hay grupos en config que no están en README

**Requisito**: Requiere el secret `ESB_ACE12_ORG_REPO_TOKEN` para acceder al repositorio de configuración.

### 6. Validar Rutas y Revisores

**Propósito**: Asegurar que los Pull Requests tengan los revisores apropiados según la ruta de despliegue.

**Reglas**:

#### 6.1. Ruta develop → quality
- Debe tener al menos un revisor de la lista de autorizados
- Revisores válidos: `DRamirezM`, `cdgomez`, `acardenasm`, `CAARIZA`

#### 6.2. Ruta quality → main
- Debe tener al menos un revisor de la lista de autorizados
- Revisores válidos: `DRamirezM`, `cdgomez`, `acardenasm`, `CAARIZA`

#### 6.3. Excepción de Emergencia
- Para rutas feature/** → develop, existe una excepción manual
- Si un comentario en el PR contiene el texto `@bot aprobar excepción`, se permite el merge sin revisor
- Esta excepción solo aplica para emergencias

## Reglas Específicas por Sección

### Resumen de Reglas DataPower

| Escenario | Regla | Resultado |
|-----------|-------|-----------|
| Sin secciones DataPower | Ninguna subsección presente | ⚠️ Warning - Recomendar indicar NA explícitamente |
| Solo DataPower Interno | Con tabla de endpoints | ✅ Válido |
| Solo DataPower Externo | Con tabla de endpoints | ✅ Válido |
| DataPower Interno | Encabezado presente, contenido vacío | ❌ Error |
| DataPower con NA | Encabezado presente, contenido "NA" | ✅ Válido |
| Ambas secciones | Ambas con NA | ✅ Válido |
| Ambas secciones | Una con tabla, otra vacía | ❌ Error |
| URL con boc200 | Cualquier URL con https://boc200 | ❌ Error |
| URL con boc201 | URLs correctas con https://boc201 | ✅ Válido |

### Resumen de Validaciones de URLs

| Componente | Patrón Requerido | Error si contiene |
|------------|------------------|-------------------|
| DataPower Desarrollo | https://boc201.des.app.bancodeoccidente.net | https://boc200 |
| DataPower Calidad Interno | https://boc201.testint.app.bancodeoccidente.net | https://boc200 |
| DataPower Calidad Externo | https://boc201.testdmz.app.bancodeoccidente.net | https://boc200 |
| DataPower Producción Interno | https://boc201.prdint.app.bancodeoccidente.net | https://boc200 |
| DataPower Producción Externo | https://boc201.prddmz.app.bancodeoccidente.net | https://boc200 |

## Ejemplos de Validación

### Ejemplo 1: README Completo Válido con DataPower

```markdown
# ESB_ACE12_MiServicio.

## INFORMACIÓN DEL SERVICIO

Este servicio implementa la funcionalidad de consulta de saldos para clientes...

### Último despliege

|CQ |JIRA | Fecha|
|---|---|---|
| CQ123 | JIRA-456 |15/03/2024|

## Procedimiento de despliegue

Aplicar MiServicio.properties a MiServicio.bar y desplegar en los grupos de ejecución:
BOGESERVICIOSWS05_SRV01 BOGESERVICIOSWS05_SRV02

## ACCESO AL SERVICIO

### DataPower Interno :
|AMBIENTE|TIPO COMPONENTE|NOMBRE WSP O MPG|DATAPOWER|ENDPOINT|
|---|---|---|---|---|
|DESARROLLO|WSP|WSMiServicio|BODPDEV|https://boc201.des.app.bancodeoccidente.net:4806/MiServicio/Port|
|CALIDAD|WSP|WSMiServicio|BODPQAS|https://boc201.testint.app.bancodeoccidente.net:4806/MiServicio/Port|
|PRODUCCION|WSP|WSMiServicio|BODPPRD|https://boc201.prdint.app.bancodeoccidente.net:4806/MiServicio/Port|

### Endpoint BUS
[tabla de endpoints...]

## CANALES - APLICACIONES
[contenido...]

## DEPENDENCIAS
[contenido...]

## DOCUMENTACION
[contenido...]

## SQL
[contenido...]
```

**Resultado**: ✅ Todas las validaciones pasan

### Ejemplo 2: README con DataPower NA (Válido)

```markdown
# ESB_ACE12_ServicioSinDataPower.

## INFORMACIÓN DEL SERVICIO

Este servicio no requiere acceso a DataPower...

### Último despliege
[tabla...]

## PROCEDIMIENTO DE DESPLIEGUE
[contenido...]

## ACCESO AL SERVICIO

### DataPower Interno :
NA

### DataPower Externo :
NA

### Endpoint BUS
[tabla de endpoints...]
```

**Resultado**: ✅ Todas las validaciones pasan

### Ejemplo 3: README con Error en Título

```markdown
# ESB_.

## INFORMACIÓN DEL SERVICIO
[contenido...]
```

**Resultado**: ❌ Error - "El título no puede ser solo 'ESB_'"

### Ejemplo 4: README con DataPower Vacío (Inválido)

```markdown
# ESB_ACE12_ServicioConErrores.

## INFORMACIÓN DEL SERVICIO
[contenido...]

## ACCESO AL SERVICIO

### DataPower Interno :
|AMBIENTE|TIPO COMPONENTE|NOMBRE WSP O MPG|DATAPOWER|ENDPOINT|
|---|---|---|---|---|


```

**Resultado**: ❌ Error - "No se encontraron filas de datos en tabla DataPower Interno"

### Ejemplo 5: README con SQL Inválido

```markdown
## SQL
Filtrar por Codigo de Operacion
```
select * from admesb.esb_log_auditoria where num_id_tipo_operacion = ''
select * from admesb.esb_log_auditoria where num_id_tipo_operacion = '99a9042'
```
```

**Resultado**: 
- ❌ Error - "Código de operación está vacío (num_id_tipo_operacion = '')"
- ❌ Error - "Código de operación contiene caracteres no numéricos"

### Ejemplo 6: README con URL Incorrecta (Inválido)

```markdown
### DataPower Interno :
|AMBIENTE|TIPO COMPONENTE|NOMBRE WSP O MPG|DATAPOWER|ENDPOINT|
|---|---|---|---|---|
|DESARROLLO|WSP|WSMiServicio|BODPDEV|https://boc200.des.app.bancodeoccidente.net:4806/MiServicio/Port|
```

**Resultado**: ❌ Error - "Los endpoints de DataPower deben comenzar con 'https://boc201' (NO 'https://boc200')"

### Ejemplo 7: README Sin Información Descriptiva

```markdown
# ESB_ACE12_MiServicio.

## INFORMACIÓN DEL SERVICIO

### Último despliege
[tabla...]
```

**Resultado**: ❌ Error - "La sección '## INFORMACIÓN DEL SERVICIO' no contiene información descriptiva antes de las subsecciones"

## Notas Importantes

1. **Case Sensitivity**: Algunas validaciones son case-sensitive (como los nombres de archivo) y otras no (como algunos títulos de sección).

2. **Orden de Secciones**: El orden de las secciones debe seguir la plantilla estándar.

3. **Formato de Tablas**: Las tablas deben usar el formato Markdown estándar con separadores `|`.

4. **URLs**: Las URLs deben ser completas y comenzar con `https://` (excepto donde se indique lo contrario).

5. **Valores NA**: Cuando se use para indicar que no aplica, se aceptan las siguientes formas: 'NA', 'N/A', o 'No Aplica'.

6. **DataPower**: Esta es una de las validaciones más críticas y con más reglas específicas - prestar especial atención a:
   - No dejar secciones vacías
   - Usar siempre `https://boc201` (nunca `boc200`)
   - Incluir al menos una subsección o indicar explícitamente NA en ambas

## Troubleshooting

### Error: "Título no puede ser solo ESB_"
**Solución**: Agregar un nombre descriptivo después de ESB_, por ejemplo: `# ESB_ACE12_MiServicio.`

### Error: "DataPower existe pero está vacía"
**Solución**: Agregar una tabla con endpoints o poner una de las siguientes opciones si no aplica:
- "NA"
- "N/A"  
- "No Aplica"

### Warning: "No se encontró acceso a DataPower"
**Solución**: Agregar las subsecciones DataPower Interno/Externo con tablas o "NA"

### Error: "Endpoint debe comenzar con https://boc201"
**Solución**: Verificar que todas las URLs de DataPower comiencen con `boc201` y no `boc200`

### Error: "Grupos de ejecución no coinciden"
**Solución**: Verificar que los grupos listados en el README coincidan exactamente con los del archivo de configuración central

## Historial de Cambios

### Corrección de validación DataPower con NA (Fecha actual)
**Problema**: El validador reportaba error "No se encontraron filas de datos en tabla DataPower" aunque el README contenía explícitamente 'NA', 'N/A' o 'No Aplica' como contenido válido.

**Causa**: La función `validate_datapower_table` se ejecutaba incluso cuando la sección contenía solo texto 'NA' sin tabla, esperando encontrar filas de tabla y fallando al no encontrarlas.

**Solución implementada**:
- Se agregó verificación previa antes de llamar a `validate_datapower_table`
- Si el contenido de DataPower Externo/Interno es solo 'NA', 'N/A' o 'No Aplica' (sin estructura de tabla), se omite la validación de tabla
- Se reconocen los tres formatos: 'NA', 'N/A', y 'No Aplica' (con variaciones de espacios)
- La validación de tabla solo se ejecuta cuando existe una estructura de tabla (encabezado con `|---|`)

**Impacto**: Los usuarios ahora pueden indicar correctamente que DataPower no aplica usando cualquiera de las tres formas aceptadas sin que el validador reporte errores falsos.

## Mantenimiento

Este documento debe actualizarse cada vez que se modifiquen las reglas en el archivo `checklist.yml`. Las validaciones se ejecutan automáticamente en cada Pull Request, garantizando el cumplimiento continuo de los estándares.

Para modificar las validaciones, editar el archivo `.github/workflows/checklist.yml` y actualizar este documento correspondientemente.
