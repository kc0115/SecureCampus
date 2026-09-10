# SC-LAB-001 - Análisis Inicial de Seguridad

### Integrantes 
- Cortés Guzmán Tania Isabel 
- Reyes Gonzalez Emmanuel 
- Romero Corral Luis Carlos 

10/Septiembre/2026

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
No es lo mismo, una amezana es el actor o situación externa que *podría* causar un daño, por ejemplo que un estudiante malicioso quiera intentar ver calificaciones o informacion ajena, y en cambio la vulnerabilidad es alguna *debilidad* que tenga el mismo sistema como el endpoint /calificaciones/125 que carece de comprobaciones de autorización. (como en el primer  ejemplo de Maria)
basicamente la amenaza aprovecha la vulnerabilidad para concretar el ataque.

### 2. ¿Puede existir una vulnerabilidad aunque todavía nadie la haya explotado?
si, ya que la vulnerabilidad es una propiedad del sistema, no depende de que alguien la use o no, como por ejemplo el caso de perfil/125 y perfil/126 es un buen ejemplo, ya que la falta de validación de autorización existe desde que se escribió el codigo, independientemente si maria fue la primera en notarlo o no  


### 3. ¿Un usuario autenticado está automáticamente autorizado para cualquier recurso?

no porque la autenticación se encargar de verificar la identidad del usuario, responde a "quien eres?" y autorizacion responde a "que tienes permitido hacer o ver?", basicamente la autorizacion determina el nivel de acceso 


### 4. ¿Qué control de los propuestos debería definirse desde requisitos o diseño? ¿Por qué?
para la seguridad la verificaión de autorización a nivel de usuario en el backend (aplicada en los escenarios de los perfiles estudiante, maestro, jefe de docencia y administrador y los privilegios de cada uno), todo esto debe definirse desde los requisitos y diseño del sistema, no agregarlo despues (security by design) 


### 5. ¿Qué activo consideran más crítico y por qué?
las credenciales de autenticación, porque son la puerta de entrada a todos los demás activos, ya que si un ataque compromete una cuenta entonces tiene acceso a un perfil y su información privada 


## 10. Cierre

### ¿Qué protegerías primero en SecureCampus y qué podría impedir que ese activo permanezca seguro?

protegeríamos primero la autenticación y el control de autorización asociado a ella, ya que es el punto que, si falla, habilita el resto de los riesgos identificados
 Lo que podría impedir que permanezca seguro es agregar nuevas funcionalidades o endpoints sin aplicar de forma consistente la verificación de autorización en el servidor, confiando erróneamente en que ocultar opciones en la interfaz es suficiente protección