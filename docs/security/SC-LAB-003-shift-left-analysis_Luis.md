# SC-LAB-003 · Costo de Corrección y Shift Left

### Integrantes
- Cortés Guzmán Tania Isabel
- Reyes Gonzalez Emmanuel
- Romero Corral Luis Carlos

15/Septiembre/2026

---

## 3. Caso guiado · Recuperación de contraseña

**RF-010:** «SecureCampus deberá permitir al usuario recuperar su contraseña». El enlace generado dura 7 días y puede reutilizarse varias veces.

| Pregunta | Respuesta del equipo |
| :--- | :--- |
| **¿Dónde se originó principalmente la omisión?** | En requisitos. RF-010 solo describe la funcionalidad ("permitir recuperar contraseña") pero no define las condiciones de seguridad del token: cuánto debe durar y si puede reutilizarse. Al no estar especificado, quedó a criterio de quien lo implementó. |
| **¿Dónde podría descubrirse?** | Podría descubrirse en diseño, al modelar el token y notar que no se definió su ciclo de vida; en pruebas de seguridad, al probar explícitamente la expiración y reutilización; o, en el peor caso, en producción, si un atacante intercepta un enlace y lo reutiliza días después. |
| **¿Qué artefactos habría que cambiar si se descubre en pruebas?** | El requisito RF-010 (agregar criterios de aceptación de seguridad), el diseño del esquema del token (expiración corta, un solo uso), el código de generación y validación del token, la tabla en base de datos (marcar el token como usado/invalidar los anteriores), los casos de prueba correspondientes y posiblemente la notificación al usuario tras el cambio de contraseña. |
| **¿Qué requisitos/criterios de seguridad faltaron?** | Expiración corta del enlace (por ejemplo 15–30 minutos en lugar de 7 días), invalidación del token tras su primer uso, invalidación de tokens anteriores al generar uno nuevo, y notificación al usuario cuando se solicite o complete un cambio de contraseña. |
| **¿Qué moverían a la izquierda?** | Definir desde el requisito (RF-010) los criterios de expiración y uso único del token como parte de los criterios de aceptación, e incluir desde el diseño pruebas de seguridad específicas para tokens (expiración, reutilización) en vez de dejarlas como una revisión posterior opcional. |

---

## 4. Reto integral · tres situaciones

| Caso | Situación |
| :--- | :--- |
| **A · Administrador** | El requisito permite consultar calificaciones sin definir condiciones. Al final se aclara que consultar y modificar requieren permisos distintos. |
| **B · Upload** | Se aceptan archivos de usuarios autenticados sin definir tipo, tamaño, nombre ni almacenamiento seguro. |
| **C · Dependencia** | Una biblioteca sin vulnerabilidades conocidas al incorporarse publica una vulnerabilidad crítica 8 meses después. Nadie la detecta por 2 meses. |

| Caso | Origen | Descubrimiento | Retrabajo/impacto | Actividad Shift Left | Control posterior |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **A** | Requisitos: el requisito inicial no distinguió entre la acción de "leer" y la de "modificar" calificaciones, tratándolas como un mismo permiso. | Diseño/Desarrollo, al implementar los permisos y notar que un solo rol cubre ambas operaciones. | Medio–Alto: hay que rediseñar el modelo de permisos separando lectura de escritura, modificar el código de autorización y, si ya hay usuarios con el permiso combinado, revisar y corregir sus accesos actuales. | Especificar desde el requisito el verbo/acción exacto (leer vs. escribir) para cada recurso sensible, usando historias de usuario con criterios de aceptación por operación, no solo por recurso. | Recertificación periódica de accesos otorgados y pruebas de autorización específicas por operación (lectura/escritura) en cada release. |
| **B** | Requisitos: falta de especificación de restricciones sobre los archivos aceptados (tipo, tamaño, nombre, almacenamiento). | Pruebas de seguridad o, en el peor caso, producción (al subirse un archivo malicioso o excesivamente grande). | Alto si se descubre en producción: requiere agregar validación en backend, revisar y posiblemente eliminar/reescanear archivos ya almacenados, y gestionar un eventual incidente si algún archivo malicioso ya circuló. | Definir desde requisitos/diseño la lista blanca de tipos de archivo permitidos, el límite de tamaño, la sanitización del nombre y un almacenamiento fuera del árbol ejecutable, validado con pruebas unitarias en desarrollo. | Escaneo de contenido/antivirus en cada carga y monitoreo de anomalías (tamaños o frecuencias inusuales) en operación. |
| **C** | No es un error de desarrollo propiamente dicho: la vulnerabilidad no existía públicamente cuando se integró la biblioteca. | Debió detectarse en operación mediante monitoreo continuo de dependencias, pero se detectó con 2 meses de retraso por falta de ese proceso. | Alto: actualizar la dependencia, verificar si fue explotada, aplicar el parche y redesplegar, y posiblemente notificar a usuarios afectados; el impacto crece cuanto más tiempo pase sin detectarse. | Mantener un inventario de dependencias (SBOM) y un proceso automatizado de escaneo de vulnerabilidades (SCA) integrado al pipeline, junto con una política de actualización periódica. | Alertas automáticas de nuevas vulnerabilidades (p. ej. Dependabot/SCA) y un proceso definido de respuesta con SLA de parcheo. |

---

## 5. Escalera de costo cualitativa

Caso elegido: **B · Upload** (archivos sin restricciones de tipo, tamaño, nombre ni almacenamiento seguro).

| Momento | ¿Qué habría que corregir/revisar? | Costo/retrabajo: Bajo/Medio/Alto + por qué |
| :--- | :--- | :--- |
| **Requisitos** | Solo hay que aclarar el criterio de aceptación (tipo, tamaño, nombre, almacenamiento) antes de que se escriba código. | **Bajo.** Es un ajuste de especificación; no existe código productivo que modificar todavía. |
| **Diseño** | Ajustar el modelo de almacenamiento y el flujo de validación de archivos antes de implementarlos. | **Bajo-Medio.** Cambia el diseño, pero aún no hay código construido ni desplegado que retrabajar. |
| **Desarrollo** | Reescribir la función de carga ya implementada, agregar las validaciones faltantes y sus pruebas unitarias. | **Medio.** Se modifica código ya construido y posiblemente se retrasan tareas que dependían de esa función. |
| **Pruebas** | Regresar a desarrollo con el hallazgo, corregir la validación y volver a ejecutar el ciclo completo de pruebas. | **Medio-Alto.** Implica retrabajo entre pruebas y desarrollo, y puede comprometer la fecha de entrega planeada. |
| **Producción** | Auditar todos los documentos ya cargados (posiblemente maliciosos), aplicar un parche urgente, coordinar con usuarios afectados y gestionar el incidente. | **Alto.** No solo se corrige código: se necesita respuesta a incidentes, comunicación y revisión retroactiva de datos ya almacenados. |

---

## 6. Pregunta con truco conceptual

**¿Puede Shift Left ayudar con una vulnerabilidad que todavía no existía públicamente cuando desarrollamos? Explique qué sí puede prepararse desde antes.**

No, Shift Left no puede prevenir directamente una vulnerabilidad que aún no ha sido descubierta ni publicada, porque no hay forma de anticipar algo desconocido en el momento de desarrollar. Sin embargo, sí puede prepararse desde antes para reducir el tiempo entre que la vulnerabilidad se hace pública y el momento en que se detecta y corrige: manteniendo un inventario actualizado de dependencias (SBOM), integrando un proceso continuo de escaneo de vulnerabilidades (SCA) que se ejecute automáticamente cuando se publiquen nuevos CVEs, definiendo desde el diseño un proceso de respuesta rápida (parcheo y despliegue ágil), y evitando dependencias poco mantenidas o innecesarias. Es decir, en este caso Shift Left no significa "prevenir lo desconocido", sino "acortar el tiempo de detección y respuesta cuando lo desconocido se vuelve conocido".

---

## 7. Reflexión

**¿Shift Left elimina la necesidad de seguridad en operación?**
No. Shift Left mueve ciertas actividades de seguridad hacia etapas más tempranas, pero no elimina la necesidad de monitoreo, detección y respuesta en producción. Siempre existen riesgos que no pueden anticiparse por completo (como una vulnerabilidad de una dependencia que se publica meses después), por lo que la seguridad debe mantenerse activa durante todo el ciclo de vida, no solo al inicio.

**¿Por qué una funcionalidad puede cumplir su requisito funcional y seguir siendo insegura?**
Porque el requisito funcional describe únicamente qué debe hacer el sistema (por ejemplo, "permitir recuperar la contraseña"), no cómo debe protegerse esa funcionalidad (expiración del enlace, uso único, límites de intentos). Una funcionalidad puede "funcionar" perfectamente desde el punto de vista del usuario y, al mismo tiempo, carecer de las condiciones de seguridad necesarias si estas nunca se especificaron como parte del requisito.

**¿Qué decisión de su equipo habría sido más barata de corregir antes?**
Definir desde el requisito RF-010 la expiración corta y el uso único del token de recuperación de contraseña habría sido la corrección más barata de anticipar: ajustarlo en requisitos es solo aclarar un criterio de aceptación, mientras que corregirlo ya en producción implicaría revisar tokens ya emitidos, posibles cuentas comprometidas y coordinar una respuesta con los usuarios afectados.