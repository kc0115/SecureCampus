# SC-LAB-002 · Mapa de Seguridad a lo largo del SDLC

### Integrantes
- Cortés Guzmán Tania Isabel
- Reyes Gonzalez Emmanuel
- Romero Corral Luis Carlos

10/Septiembre/2026

---

## 5. Actividad guiada · Calificaciones

**Caso:** un estudiante autenticado puede cambiar `/calificaciones/125` por `/calificaciones/126` y consultar calificaciones ajenas.

| Fase | ¿Qué debería hacerse? | Control / evidencia |
| :--- | :--- | :--- |
| **Requisitos** | Definir explícitamente el requisito de control de acceso a nivel de objeto: "un estudiante solo puede consultar sus propias calificaciones; un profesor solo las de sus grupos asignados". No dejarlo implícito en "el usuario debe estar autenticado". | Historia de usuario con criterio de aceptación de autorización (no solo de autenticación) en el documento de requisitos. |
| **Diseño** | Modelar la relación estudiante-calificación-grupo en el esquema de datos y definir el mecanismo de autorización (RBAC/ABAC) que valide esa relación en cada consulta, no solo el rol del usuario. | Diagrama de control de acceso / matriz de permisos por rol y por recurso. |
| **Desarrollo** | Implementar en el backend la verificación de que el `id` del usuario en sesión coincide con el propietario del recurso (o tiene permiso explícito) antes de responder; evitar exponer identificadores secuenciales predecibles sin control adicional. | Middleware/función de autorización en el código; revisión de código (code review) enfocada específicamente en IDOR. |
| **Pruebas** | Ejecutar pruebas de control de acceso: intentar acceder a `/calificaciones/126` con la sesión del estudiante 125 y verificar que el sistema lo rechace (403), no que simplemente lo oculte en la interfaz. | Casos de prueba de IDOR documentados; resultados de pruebas manuales o automatizadas (SAST/DAST) sobre el endpoint. |
| **Despliegue** | Verificar que las reglas de autorización estén activas en el ambiente productivo (y no solo en desarrollo), y que no existan endpoints de depuración expuestos que salten la validación. | Checklist de despliegue seguro; gate en el pipeline que bloquea el release si las pruebas de autorización fallan. |
| **Operación/Mantenimiento** | Monitorear y registrar accesos denegados o patrones anómalos (un mismo usuario probando múltiples IDs consecutivos) para detectar intentos de IDOR en curso. | Logs de auditoría con quién/cuándo/qué recurso; alertas ante patrones de enumeración de IDs. |

---

## 6. Reto por equipo

| Escenario | Situación |
| :--- | :--- |
| **A · Documentos** | Un estudiante intenta descargar el documento de otro usuario modificando un identificador. |
| **B · Token** | Un desarrollador intenta incluir un token dentro de un commit. |
| **C · Profesor** | Un profesor intenta modificar calificaciones de un grupo no asignado. |
| **D · Login** | Una cuenta registra 100 intentos fallidos de autenticación en 10 minutos. |

| Esc. | Requisitos | Diseño | Desarrollo | Pruebas | Despliegue | Operación |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **A** | Requisito de que solo el propietario o un rol autorizado pueda descargar un documento. | Identificadores de documento no predecibles (UUID) + control de acceso por propietario/rol en el modelo de datos. | Validar en el backend que el usuario en sesión es el propietario (o tiene permiso) antes de servir el archivo; usar URLs/tokens de descarga firmados y con expiración. | Pruebas de IDOR sobre el endpoint de descarga usando sesiones de distintos usuarios. | Confirmar que el almacenamiento de documentos no sea de acceso público directo (bucket/carpeta privada, no indexable). | Monitoreo de descargas inusuales (mismo usuario accediendo a muchos documentos distintos en poco tiempo). |
| **B** | Requisito no funcional: ningún secreto (token, contraseña, llave) debe almacenarse en el repositorio. Política de manejo de secretos. | Diseño de gestión de secretos vía variables de entorno o un vault, separado del código fuente. | Uso de pre-commit hooks (p. ej. git-secrets) que bloqueen commits con patrones de credenciales; `.gitignore` para archivos de configuración sensibles. | Escaneo automático de secretos (secret scanning) en el pipeline de CI antes de permitir el merge. | Gate en el pipeline que impide el despliegue si se detecta un secreto expuesto. | Monitoreo continuo del repositorio (alertas de secret scanning) y rotación inmediata de cualquier credencial que se filtre. |
| **C** | Requisito de autorización a nivel de objeto para la captura de calificaciones: solo sobre grupos asignados al profesor. | Modelo de datos que relacione explícitamente profesor–grupo–materia, validado en cada operación de escritura. | Validación en el backend de que el grupo enviado pertenece al profesor autenticado antes de permitir capturar/editar calificaciones. | Pruebas de autorización intentando modificar un grupo ajeno con una cuenta de profesor válida. | Verificar que la asignación profesor–grupo se sincronice correctamente en producción tras cada alta o cambio de grupo. | Auditoría de cada cambio de calificación (quién, cuándo, valor anterior/nuevo) y alertas ante cambios fuera de los grupos asignados. |
| **D** | Requisito de límite de intentos fallidos, bloqueo/retardo progresivo y política de contraseñas robustas. | Diseño del mecanismo de rate limiting y bloqueo de cuenta/IP; diseño opcional de MFA. | Implementación del contador de intentos fallidos, retardo exponencial o captcha, y notificación al usuario ante intentos sospechosos. | Pruebas simuladas de fuerza bruta para validar que el bloqueo se active tras N intentos. | Configurar rate limiting también a nivel de infraestructura (WAF/proxy), no solo en la aplicación. | Monitoreo de intentos fallidos masivos, alertas al equipo de seguridad y revisión periódica de logs de autenticación. |

---

## 7. Clasificación conceptual

| Decisión | Secure SDLC / By Design / By Default / Shift Left | Justificación |
| :--- | :--- | :--- |
| **1.** Validar en el backend, desde el modelo de datos, que la relación profesor–grupo o estudiante–calificación exista antes de permitir el acceso (escenarios de calificaciones y documentos). | **Security by Design** | La protección no se agrega después como un parche; se integra en la arquitectura y en el modelo de datos desde el diseño, de modo que sea estructuralmente imposible acceder a un recurso ajeno sin pasar por la validación. |
| **2.** El bloqueo progresivo de intentos fallidos de login se activa automáticamente para toda cuenta nueva, sin que el administrador tenga que configurarlo. | **Security by Default** | La configuración segura viene activada de fábrica; el usuario u operador no necesita habilitar nada adicional para tener esa protección mínima. |

---

## 8. Reflexión

**¿Qué riesgo de SC-LAB-001 necesitó controles en más fases?**
El riesgo de IDOR en perfiles/calificaciones (SC-LAB-001, caso de María) es el que requiere controles en más fases: hay que definirlo desde requisitos, modelarlo en diseño, implementarlo en desarrollo, verificarlo con pruebas específicas de autorización, confirmarlo en el despliegue y seguir monitoreándolo en operación mediante auditoría. No basta con "arreglarlo en el código"; si no se sostiene en todas las fases, vuelve a aparecer en cuanto se agregue un nuevo endpoint.

**¿Qué habría ocurrido si el equipo hubiera esperado hasta pruebas?**
El diseño del sistema probablemente ya habría fijado un modelo de datos y de endpoints sin la relación explícita usuario-recurso necesaria para autorizar correctamente. Corregirlo en pruebas implicaría regresar a rediseñar esa relación y reescribir el código de varios módulos ya construidos, en vez de solo agregar una validación puntual, lo que representa mucho más retrabajo que haberlo definido desde el requisito.

**¿Qué control depende de una regla de negocio y cuál puede automatizarse?**
La relación profesor–grupo–materia (quién tiene permitido capturar calificaciones de qué grupo) depende de una regla de negocio, porque la define la institución (asignaciones académicas) y puede cambiar cada semestre. En cambio, el bloqueo tras múltiples intentos fallidos de login o el escaneo de secretos en cada commit son controles técnicos que pueden automatizarse completamente en el pipeline sin depender de una decisión de negocio caso por caso.