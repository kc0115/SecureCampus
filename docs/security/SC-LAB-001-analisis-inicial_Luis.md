# SC-LAB-001 - Análisis Inicial de Seguridad

## 4. Actividad guiada: consulta de perfiles

| Elemento | Respuesta del equipo | Justificación |
| :--- | :--- | :--- |
| **Activo** | Información personal y académica del perfil del estudiante. | Es el recurso con valor dentro del sistema que requiere protección rigurosa de su confidencialidad. |
| **Amenaza** | Un usuario legítimo (estudiante) actuando con curiosidad o intención maliciosa. | María ya cuenta con credenciales válidas y acceso a la red, representando una amenaza interna. |
| **Vulnerabilidad** | Falta de control de acceso a nivel de objeto (*Insecure Direct Object Reference* - **IDOR**) en el backend. | El sistema confía ciegamente en el parámetro enviado en la URL sin validar la autorización sobre el recurso solicitado. |
| **Ataque** | Manipulación de parámetros (*Parameter Tampering*). | Modificación directa e intencional del identificador en la petición HTTP enviada al servidor. |
| **Impacto** | Brecha de confidencialidad y violación a la privacidad. | Exposición no autorizada de Información de Identificación Personal (PII) e historial académico de otros alumnos. |
| **Riesgo** | **Alto** | Alta probabilidad de ocurrencia debido a la nula complejidad técnica para explotarlo, sumado a un impacto directo y severo a la privacidad. |
| **Control** | Validación estricta de autorización en el servidor (ABAC / RBAC). | El backend debe verificar siempre que el ID del usuario en la sesión activa posea permisos explícitos sobre el recurso solicitado antes de procesar la respuesta. |

---

## 5. Reto por equipo: Escenarios de SecureCampus

| Escenario | Activo | Amenaza | Vulnerabilidad | Ataque | Impacto | Control |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Calificaciones** | Base de datos de registros y promedios académicos. | Estudiante intentando alterar su promedio final; atacante manipulando notas. | Insuficiente validación de roles en la API de escritura. | Elevación de privilegios o Inyección SQL enviando una petición de actualización no autorizada. | Pérdida de integridad de los datos académicos y devaluación de la fiabilidad institucional. | Implementar control RBAC estricto en el servidor. Solo la sesión validada con rol "Profesor" asignado a ese grupo específico puede modificar calificaciones. |
| **2. Documentos** | Archivos, identificaciones y comprobantes digitales subidos a la plataforma. | Usuario malintencionado intentando comprometer el servidor o leer archivos de otros. | Falta de validación del tipo de archivo y ausencia de autenticación requerida en descargas. | Subida de archivos maliciosos o salto de directorio (Path Traversal). | Compromiso del sistema si un malware se ejecuta, o fuga masiva de documentos sensibles. | Almacenamiento de documentos en contenedores privados, exigiendo token de acceso y validación de permisos para su descarga. |
| **3. Autenticación** | Credenciales de usuario y tokens de administración de sesiones. | Atacante automatizado intentando descubrir contraseñas o interceptar la red. | Almacenamiento de credenciales en texto plano en las tablas de la base de datos. | Ataque de Fuerza Bruta, *Credential Stuffing* o secuestro de sesión (*Session Hijacking*). | Suplantación de identidad y acceso ilimitado a operaciones críticas de administradores. | Hashing criptográfico fuerte para contraseñas, prohibiendo credenciales en texto plano. Implementar políticas de bloqueo temporal (rate limiting). |
| **4. Elección del equipo (Jefe de Docencia)** | Estructura de cargas académicas: alta de materias, creación y asignación de grupos. | Profesor o administrador no autorizado buscando alterar la estructura académica y autoasignarse grupos. | Omisión en la segregación de funciones y falta de validación de este rol específico de alta jerarquía. | Explotación del control de acceso funcional (Broken Access Control). | Desorganización institucional severa y apertura de vías para la captura ilícita de calificaciones. | Reconocer explícitamente al "Jefe de docencia" como actor en el backend para la creación y asignación de grupos a profesores. Registrar estas operaciones en un log de auditoría. |

---

## 6. Preguntas de reflexión

### 1. ¿Una amenaza y una vulnerabilidad son lo mismo? Explica con un ejemplo de SecureCampus.
> **No.** 
> - Una **amenaza** es el agente o circunstancia externa que puede materializar un daño, como un estudiante curioso intentando alterar recursos. 
> - La **vulnerabilidad** es la debilidad técnica intrínseca en el sistema, como el endpoint `/calificaciones/125` que carece de comprobaciones de autorización. 
> 
> *Conclusión:* La amenaza aprovecha la vulnerabilidad para concretar el ataque.

---

### 2. ¿Puede existir una vulnerabilidad aunque todavía nadie la haya explotado?
> **Totalmente.** Un error en la validación de autorización existe desde el momento en que el código es escrito y desplegado, independientemente de si un atacante ya ha localizado o abusado de esa falla. La vulnerabilidad es una condición latente del sistema.

---

### 3. ¿Un usuario autenticado está automáticamente autorizado para cualquier recurso?
> **Absolutamente no.** 
> - La **autenticación** se encarga exclusivamente de verificar la identidad (responder a la pregunta: *¿quién eres?*).
> - La **autorización** determina el nivel de acceso sobre un objeto (responder a la pregunta: *¿qué puedes hacer con este recurso?*).
> 
> Se debe evitar el supuesto de que "estar autenticado" otorga automáticamente permisos para cualquier operación.

---

### 4. ¿Qué control de los propuestos debería definirse desde requisitos o diseño? ¿Por qué?
> El modelo de **Control de Acceso Basado en Roles (RBAC)** y la política de **no almacenar credenciales en texto plano**. 
> 
> **Por qué:** Esto es vital porque la seguridad debe incorporarse al ciclo de vida del software desde el inicio (*Security by Design*), integrándose en las decisiones arquitectónicas tempranas en lugar de añadirse como un parche superficial al final del desarrollo.

---

### 5. ¿Qué activo consideran más crítico y por qué?
> La base de datos del **Módulo de Autenticación, Roles y Auditoría**. 
> 
> **Por qué:** Si un atacante compromete los mecanismos que validan identidades y roles, obtendrá control total para leer perfiles, alterar actas académicas y borrar el rastro de sus operaciones en los registros de auditoría.

---

## 10. Cierre

### ¿Qué protegerías primero en SecureCampus y qué podría impedir que ese activo permanezca seguro?

* **¿Qué proteger primero?**  
  El primer mecanismo a salvaguardar es la capa de **Validación de Identidad y Autorización desde el lado del Servidor (Backend)**.

* **¿Qué podría impedir que permanezca seguro?**  
  La suposición errónea de que la seguridad consiste únicamente en validar en la interfaz gráfica (frontend), olvidando la regla fundamental de que **el servidor siempre debe verificar de manera independiente cada petición**.