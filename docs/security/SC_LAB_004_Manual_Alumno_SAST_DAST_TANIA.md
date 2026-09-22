# Evidencia de Laboratorio: SC-LAB-004 Análisis SAST y DAST

## 1. Objetivo
Analizar la miniaplicación de SecureCampus utilizando técnicas de análisis de seguridad estático (SAST) y dinámico (DAST), interpretar la evidencia, corregir las vulnerabilidades deliberadas (SQLi y XSS) y comprobar las correcciones mediante un reanálisis.

## 2. Entorno
* **Sistema:** Windows + PowerShell + VS Code
* **Aplicación:** Python 3.14.x, Flask 3.x
* **Herramientas de Seguridad:** Semgrep Community (SAST) y OWASP ZAP (DAST) ejecutados vía contenedores Docker.

## 3. Parte A: Análisis SAST

### 3.1 Análisis Humano Previo
Revisión manual del archivo `src/app.py`:
* **¿Qué dato controla el usuario?** El parámetro `nombre` ingresado en la búsqueda.
* **¿A dónde llega ese dato?** A la consulta SQL y a la plantilla HTML de respuesta.
* **¿Qué riesgo observas?** Inyección SQL (SQLi) y Cross-Site Scripting (XSS) reflejado.
* **¿Qué control propondrías?** Consultas preparadas (parametrizadas) para el SQL y escape de variables en el HTML.

### 3.2 Ejecución y Hallazgo Semgrep
**Comando ejecutado:**
```bash
docker run --rm -v "${PWD}:/src" semgrep/semgrep semgrep scan --config auto /src/src
```

**Evidencia:**
* **Regla/hallazgo:** Uso inseguro de concatenación en cadenas SQL (Posible Inyección SQL).
* **Archivo/línea:** `src/app.py` (en la declaración y ejecución del `cursor.execute`).
* **Coincidencia:** Confirma la sospecha del análisis manual sobre la vulnerabilidad de la base de datos.

### 3.3 Corrección y Reanálisis SAST
Se reemplazó la concatenación por una consulta parametrizada:
```python
consulta = (
    "SELECT id, nombre, correo "
    "FROM estudiantes "
    "WHERE nombre = ?"
)
cursor.execute(consulta, (nombre,))
```
Tras ejecutar Semgrep nuevamente, se obtuvieron **0 findings**. *Nota: Esto significa que la regla de SQLi ya no se activa, no que la aplicación sea 100% segura.*

---

# 4.1 Analiza primero como desarrollador
### ¿Qué dato controla el usuario?
El parámetro de entrada nombre ingresado a través de la consola mediante input("Nombre del estudiante: ").   

### ¿A dónde llega ese dato?
Se pasa como argumento a la función buscar_estudiante(nombre) y se concatena directamente dentro de la cadena de consulta SQL.  

### ¿Qué riesgo observas?
Vulnerabilidad de Inyección SQL (SQLi). Al concatenar directamente la entrada del usuario ("WHERE nombre = '" + nombre + "'"), un atacante podría alterar la lógica de la consulta.  

### ¿Qué control propondrías?
Implementar consultas parametrizadas (Prepared Statements) pasando los argumentos como una tupla al ejecutor del cursor (cursor.execute(consulta, (nombre,))). 

# 4.2 Ejecuta Semgrep
## Regla/hallazgo
python.lang.security.audit.sqli.sqlite-string-formatting (o regla equivalente de detección de SQL Injection en SQLite por formateo/concatenación de cadenas).

## Archivo/línea
src/app.py, líneas 6-9 (dentro de la definición de la variable consulta en buscar_estudiante). 

## ¿Qué evidencia aporta?
Señala que una variable proveniente de entrada no confiable se está concatenando directamente en una sentencia SQL sin sanitización ni parametrización.

## ¿Coincide con tu análisis humano?
Sí. La herramienta confirmó exactamente el punto de inyección SQL identificando la fuente (entrada nombre) y el sumidero/sink (cursor.execute)

---
## 5. Comparación Final SAST vs DAST

| Criterio | SAST | DAST |
| :--- | :--- | :--- |
| **Objeto** | Código fuente | Aplicación en ejecución |
| **Necesita ejecutar app** | No | Sí |
| **Perspectiva** | Interna/estática | Externa/dinámica |
| **Evidencia del lab** | Construcción SQL insegura | XSS y configuración HTTP |
| **Fortaleza** | Detecta patrones/rutas en código | Observa comportamiento real expuesto |
| **Límite** | No garantiza lógica de negocio | No ve todo el código ni todas las rutas |

---

# Preguntas de Reflexión

### 1. ¿Por qué 0 findings en SAST no equivale a una aplicación segura?
 Un resultado de "0 findings" en SAST solo significa que el analizador no encontró patrones inseguros basados en las reglas y firmas que tiene configuradas en ese momento. SAST no puede evaluar la lógica de negocio, errores de configuración en producción, problemas de autenticación complejas, ni vulnerabilidades de terceros no mapeadas. Además, no analiza el comportamiento de la aplicación en ejecución.

### 2. ¿Por qué un WARN de ZAP debe validarse antes de declararlo vulnerabilidad?
ZAP realiza un análisis automatizado que puede generar **falsos positivos** (alertas sobre comportamientos que parecen vulnerables pero no representan un riesgo real dentro del contexto de la aplicación) o reportar advertencias de buenas prácticas (como cabeceras de seguridad faltantes) que no necesariamente son explotables. La validación manual permite confirmar el impacto real, si la falla es explotable en ese entorno específico y evitar trabajar en falsas alarmas.

### 3. ¿Qué diferencia observaste entre baseline y active scan?
Baseline (Escaneo Pasivo):Examina las peticiones y respuestas HTTP normales sin modificar el tráfico ni enviar cargas útiles (payloads) maliciosas. Detecta principalmente faltas de cabeceras de seguridad o cookies inseguras.
Active Scan (Escaneo Activo): Interactúa activamente con la aplicación intentando manipular entradas, enviar payloads de ataque (por ejemplo, scripts maliciosos o inyecciones SQL) e interpretar las respuestas para descubrir vulnerabilidades dinámicas como XSS Reflejado o SQLi.

### 4. ¿Por qué 200 OK no descarta una vulnerabilidad?
Un código de estado `200 OK` simplemente indica que el servidor procesó y respondió la solicitud exitosamente. Una vulnerabilidad (como Cross-Site Scripting reflejado o una inyección SQL) puede ejecutarse correctamente y devolver una página web válida con código HTTP 200. El riesgo reside en la **presencia de carga maliciosa no procesada/satinizada en el cuerpo de la respuesta**, no en el código de estado del protocolo HTTP.

### 5. ¿Qué aprendiste del hecho de tener que reiniciar Flask antes del retest?
Aprendí que si la aplicación web en ejecución no está configurada en modo de recarga automática (*auto-reload* o *debug*), los cambios aplicados en el código fuente no se cargan dinámicamente en la memoria del proceso activo. Si no se reinicia el servidor, el escaneo de retesting se ejecutará contra la versión antigua y vulnerable aún alojada en memoria, dando un resultado falso de que la corrección no funcionó.

### 6. ¿Qué problema de autorización podría seguir existiendo aunque SAST y DAST no lo reporten?
 Podrían existir fallas de **Control de Acceso Roto (Broken Access Control)** o **BOLA / IDOR (Insecure Direct Object References)**. Las herramientas automatizadas no comprenden las reglas del negocio ni la matriz de permisos de la aplicación (por ejemplo, si el estudiante A puede ver la información personal del estudiante B simplemente cambiando el ID en la URL). Estos problemas requieren pruebas lógicas y revisiones humanas de diseño para ser detectados.


