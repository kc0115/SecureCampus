# SC-LAB-001 - Análisis Inicial de Seguridad

## 4. Actividad guiada: consulta de perfiles

**Caso:** María inicia sesión con el perfil 125. Al observar la URL, cambia manualmente `/perfil/125` por `/perfil/126`. El sistema devuelve información de otro estudiante.

| Elemento | Respuesta del equipo | Justificación |
| :--- | :--- | :--- |
| **Activo** | Información personal y académica del perfil del estudiante. | Es el recurso con valor dentro del sistema que requiere protección rigurosa de su confidencialidad, su exposicion no autorizada afecta directamente a la privacidad. |
| **Amenaza** | Un usuario autenticado (estudiante) actuando con curiosidad o intención maliciosa. | María ya cuenta con credenciales válidas y acceso a la red, representando una amenaza interna. |
| **Vulnerabilidad** | Falta de control de acceso a nivel de objeto (*Insecure Direct Object Reference* - **IDOR**) en el backend. | El sistema confía ciegamente en el parámetro enviado en la URL sin validar la autorización sobre el recurso solicitado. |
| **Ataque** | Manipulación de parámetros: el atacante cambia manualmente el identificador en la URL (`/perfil/126`, `/perfil/127`, …) para acceder a otros perfiles. | Modificación directa e intencional del identificador en la petición HTTP enviada al servidor. |
| **Impacto** | Brecha de confidencialidad, posible violación de normativas de protección de datos y pérdida de confianza en la plataforma. | Exposición no autorizada de Información de Identificación Personal (PII) e historial académico de otros alumnos. |
| **Riesgo** | **Alto**, la proobabilidad de explotacion es alta y el impacto se vuelve un tanto significativo. | Alta probabilidad de ocurrencia debido a la nula complejidad técnica para explotarlo, sumado a un impacto directo y severo a la privacidad. |
| **Control** | Validación estricta de autorización en el servidor (ABAC / RBAC). | El backend debe verificar siempre que el ID del usuario en la sesión activa posea permisos explícitos sobre el recurso solicitado antes de procesar la respuesta. |

---

## 5. Reto por equipo — Matriz de escenarios

Analicen al menos cuatro escenarios diferentes de SecureCampus. Deben incluir obligatoriamente: calificaciones, documentos y autenticación; el cuarto escenario lo elige el equipo (roles/permisos, solicitudes, logs, perfiles u otro módulo aprobado).

| Escenario | Activo | Amenaza | Vulnerabilidad | Ataque | Impacto | Control |
|---|---|---|---|---|---|---|
| **1.Calificaciones** | Registros de calificaciones de estudiantes por grupo/materia. | Un profesor (u otro usuario autenticado) que modifica calificaciones de grupos que no tiene asignados. | El backend no valida que el grupo pertenezca al profesor autenticado antes de permitir la captura/edición de calificaciones. | El profesor envía directamente una solicitud con el ID de un grupo ajeno para capturar o alterar calificaciones. | Calificaciones alteradas, afectando la credibilidad del sistema y de la institución. | Verificación de autorización a nivel de objeto en cada operación de escritura más registro de auditoría de cada cambio de calificación (quién, cuándo, valor anterior/nuevo). |
| **2. Documentos** | Documentos oficiales y personales cargados por estudiantes. | Un usuario que intenta descargar o visualizar documentos que no le pertenecen. | Los documentos se referencian por un identificador o URL predecible/expuesta, sin control de acceso por propietario, o sin validar el tipo/contenido al subirlos. | Manipulación del identificador del documento en la URL de descarga, o carga de un archivo malicioso disfrazado de documento válido. | Fuga de documentos sensibles de terceros, o ejecución de contenido malicioso si el sistema no valida el archivo subido, comprometiendo confidencialidad y disponibilidad. | Control de acceso por propietario/rol en cada descarga, validación estricta de tipo y contenido de archivo en la carga. |
| **3. Autenticación** | Credenciales y sesiones de los usuarios. | Un atacante externo que intenta obtener acceso no autorizado probando credenciales. | Ausencia de límite de intentos de inicio de sesión y/o políticas débiles de contraseñas. | Ataque de fuerza bruta contra el formulario de login, probando combinaciones de usuario/contraseña de forma automatizada. | Toma de control de cuentas, permitiendo suplantación de identidad y acceso a todo lo que el rol comprometido puede ver o modificar. | Bloqueo progresivo o retardo tras intentos fallidos, autenticación multifactor, políticas de contraseña robustas y notificación al usuario ante intentos sospechosos. |
| **4.Roles/permisos** | Configuración de roles y permisos que determina qué puede hacer cada tipo de usuario. | Un usuario con privilegios bajos que intenta ejecutar una acción reservada a administradores. | Falta de verificación de rol/permiso en el servidor para operaciones sensibles; la interfaz solo oculta opciones administrativas en el cliente, pero el backend las acepta. | El usuario invoca directamente el endpoint administrativo sin pasar por la interfaz gráfica que oculta esa opción. | Un estudiante o profesor podría obtener capacidades de administrador, comprometiendo la integridad de todo el sistema. | Aplicar control de acceso basado en roles verificado siempre del lado del servidor para cada endpoint sensible y principio de mínimo privilegio por defecto. |

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