# Sistema de Monitoreo Marítimo -- Modelo Formal en VDM++

## Integrantes:
## Diego Nova Rosas
## Angélica Castillo Tovar
## Renzo Murillo Álvarez
## Marcelo Silva Cabrera

Este proyecto presenta un **modelo formal** de un sistema de monitoreo
marítimo utilizando **VDM++**, aplicado en el contexto del curso
*Métodos Formales de Ingeniería de Software*.\
El objetivo es modelar, con rigor matemático, la detección de
infracciones marítimas mediante zonas de veda, zonas prohibidas,
evidencias geográficas y embarcaciones registradas.

El sistema incorpora **precondiciones**, **postcondiciones**,
**invariantes**, **tipos abstractos** y validaciones formales que
permiten verificar la corrección del modelo.

------------------------------------------------------------------------

## 1. Clase `Embarcacion`

Representa la información básica de una embarcación dentro del sistema.

### **Atributos**

-   `matricula`: identificador único.
-   `tipo`: tipo de embarcación.
-   `bandera`: país o bandera.
-   `licenciaVigente`: si la embarcación tiene licencia válida.

### **Principales reglas formales**

**Precondición del constructor**

``` vdm
pre mat <> [] and tip <> [] and band <> []
```

Los atributos textuales deben ser no vacíos.

**Postcondición**

``` vdm
post matricula = mat and tipo = tip and bandera = band;
```

### **Operaciones**

-   `getMatricula()`, `getTipo()`, `getBandera()`
-   `tieneLicencia()`

------------------------------------------------------------------------

## 2. Clase `ZonaVeda`

Zona restringida por temporada, con latitud y longitud delimitadas.

### **Atribututos**

Incluye: - nombre - latitudMin / latitudMax - longitudMin /
longitudMax - fechaInicio / fechaFin - activa

### **Validaciones del constructor**

``` vdm
pre nom <> [] and 
    latMin >= -90 and latMax <= 90 and latMin < latMax and
    lonMin >= -180 and lonMax <= 180 and lonMin < lonMax
post nombre = nom and activa = true;
```

### **Operaciones**

-   `activarVeda()` / `desactivarVeda()`
-   `estaActiva()`
-   `contieneUbicacion(lat, lon)`
-   `getCoordenadas()`

------------------------------------------------------------------------

## 3. Clase `ZonaProhibida`

Área restringida permanentemente o por parámetros específicos.

### **Atributos**

-   nombre
-   límites geográficos
-   `permanente`: indica si la restricción es de carácter fijo.

### **Validaciones**

``` vdm
pre nom <> [] and 
    latMin >= -90 and latMax <= 90 and latMin < latMax and
    lonMin >= -180 and lonMax <= 180 and lonMin < lonMax
```

### **Operaciones**

-   `contieneUbicacion(lat, lon)`
-   `getNombre()`
-   `esPermanente()`

------------------------------------------------------------------------

## 4. Clase `Evidencia`

Registra datos geográficos capturados por sensores: ubicación,
embarcación vinculada, timestamp, etc.

### **Atributos**

-   id
-   embarcacionId
-   latitud / longitud
-   timestamp
-   fuente
-   embarcación (opcional)

### **Reglas formales**

``` vdm
pre eid <> [] and embId <> [] and 
    lat >= -90 and lat <= 90 and 
    lon >= -180 and lon <= 180 and
    ts <> [] and fue <> []
post id = eid and latitud = lat and longitud = lon;
```

------------------------------------------------------------------------

## 5. Clase `Alerta`

Representa una infracción detectada por el sistema.

### **Tipo definido con invariante**

``` vdm
TipoInfraccion = seq of char
inv t == t in set {"VEDA_TEMPORAL", "ZONA_PROHIBIDA", "DOCUMENTO_INVALIDO", "SIN_LICENCIA"};
```

### **Constructor**

``` vdm
pre aid <> [] and 
    tipo in set {"VEDA_TEMPORAL", "ZONA_PROHIBIDA", "DOCUMENTO_INVALIDO", "SIN_LICENCIA"} and
    conf >= 0 and conf <= 1
post id = aid and nivelConfianza = conf;
```

### **Operaciones**

-   `notificar(canal)` -- marca la alerta como enviada.
-   Métodos getter: tipo, justificación, nivel de confianza, etc.

------------------------------------------------------------------------

## 6. Clase `SistemaMonitoreo`

Modelo central que gestiona todos los elementos del sistema.

### **Atributos**

-   `embaracaciones`
-   `evidencias`
-   `alertas`
-   `zonasVeda`
-   `zonasProhibidas`

Inicializados siempre como listas vacías.

------------------------------------------------------------------------

## 6.1 Registro de elementos

El sistema permite:

-   Registrar embarcaciones\
-   Registrar zonas de veda\
-   Registrar zonas prohibidas\
-   Recibir evidencias

Todas las operaciones agregan elementos a secuencias mediante
concatenación (`^`).

------------------------------------------------------------------------

## 6.2 Procesamiento de evidencia

La operación **procesarEvidencia(ev)** realiza tres validaciones
formales:

### **1. Verificar zonas de veda activas**

Si la ubicación de la evidencia cae dentro de una veda activa →\
 genera una alerta `"VEDA_TEMPORAL"`.

### **2. Verificar zonas prohibidas**

Si cae dentro de una zona prohibida →\
 genera una alerta `"ZONA_PROHIBIDA"`.

### **3. Verificar licencia de la embarcación**

Si la embarcación no tiene licencia →\
 se genera `"SIN_LICENCIA"`.

### **Comportamiento adicional**

-   Convierte latitud/longitud a texto ("Norte/Sur", "Este/Oeste").
-   Cada alerta creada se almacena en `alertas`.
-   Si no hay infracción → retorna `nil`.

------------------------------------------------------------------------

## 6.3 Consultas disponibles

-   `obtenerAlertas()`
-   `obtenerAlertasPorTipo(tipo)`
-   `contarAlertas()`
-   `listarZonasVeda()`
-   `listarZonasProhibidas()`

------------------------------------------------------------------------

# Conclusión

Este modelo en VDM++ demuestra:

-   Aplicación de **métodos formales** a través de invariantes,
    precondiciones y postcondiciones.
-   Diseño orientado a objetos verificado.
-   Validación rigurosa de datos geográficos.
-   Encapsulamiento formal entre clases.
-   Flujo completo de detección de infracciones marítimas basado en
    lógica declarativa.

Es un ejemplo representativo de cómo los métodos formales permiten
**crear sistemas robustos, verificables y libres de ambigüedades** en su
especificación.
