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

* **¿Qué proteger primero?**  
  El primer mecanismo a salvaguardar es la capa de **Validación de Identidad y Autorización desde el lado del Servidor (Backend)**.

* **¿Qué podría impedir que permanezca seguro?**  
  La suposición errónea de que la seguridad consiste únicamente en validar en la interfaz gráfica (frontend), olvidando la regla fundamental de que **el servidor siempre debe verificar de manera independiente cada petición**.