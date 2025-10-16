# REQUERIMIENTOS DEL SISTEMA

## 1. Descripción del Cliente y Problema Principal

**Cliente:** Taller mecánico automotriz (negocio real - padre del equipo).

**Problema Principal:**
El taller mecánico maneja todas las órdenes de trabajo de forma manual en cuadernos y papeles sueltos, lo que genera pérdida de información y desorganización. No hay forma de enviar documentación formal a los clientes por correo electrónico, obligándolos a llamar constantemente para conocer el estado de sus vehículos.

**Necesidad:**
Crear un sistema digital que permita gestionar presupuestos y órdenes de trabajo de manera eficiente, con notificación automática por correo electrónico al cliente, mejorando la comunicación y profesionalizando el servicio.

## 2. Plazo y Alcance del Proyecto

### Plazo de Desarrollo:
**4 semanas** para completar el MVP funcional.

### Alcance del Proyecto:

**Incluido en el MVP:**
- Sistema de autenticación con JWT.
- Gestión completa de clientes (crear, editar, eliminar, listar, buscar).
- **Gestión de presupuestos (crear, editar, enviar por correo, aprobar/rechazar).**
- **Creación automática de orden de trabajo al aprobar presupuesto.**
- Gestión de órdenes de trabajo (listar, editar, cambiar estado).
- Asignación de mecánicos a órdenes de trabajo.
- Estados de órdenes: En Reparación, Listo, Entregado.
- **Envío automático de correo electrónico al cliente cuando la orden cambia a estado "Listo".**
- Base de datos MongoDB.
- Cache con Redis para consultas frecuentes.
- Proyecto dockerizado con documentación completa.

**Excluido del MVP (versiones futuras):**
- Interfaz gráfica web completa (solo API REST).
- Gestión de inventario y repuestos.
- Roles adicionales (Cliente con login, Recepcionista).
- Reportes y dashboards con gráficos.
- Sistema de facturación.
- Notificaciones Correo o WhatsApp.
- Firma digital de presupuestos/órdenes.
- Historial detallado por vehículo (se puede agregar después).

### Criterios de Éxito:
- El sistema permite crear presupuestos y enviarlos por correo.
- Al aprobar un presupuesto, se crea automáticamente una orden de trabajo.
- Los clientes reciben automáticamente un correo cuando su vehículo está listo.
- Las órdenes tienen estados claros y se puede hacer seguimiento.
- Los mecánicos pueden ver las órdenes asignadas.
- El proyecto se ejecuta correctamente con docker-compose.
- La documentación permite a cualquier desarrollador levantar el proyecto.

## 3. Lista de Usuarios del Sistema

- Administrador (Dueño del taller)
- Mecánico

## 4. Usuarios y Permisos según Rol

**Administrador:**
- Acceso completo al sistema.
- Crear, editar y eliminar clientes.
- **Crear, editar y enviar presupuestos.**
- **Aprobar/rechazar presupuestos.**
- Gestionar órdenes de trabajo.
- Asignar mecánicos a órdenes.
- Cambiar estado de órdenes.
- Ver historial completo de presupuestos y órdenes.

**Mecánico:**
- Ver órdenes asignadas a él.
- Actualizar estado de sus órdenes.
- Agregar notas y observaciones.

## 5. Funciones Indispensables por Perfil

**Administrador (orden de prioridad):**
1. Autenticación y acceso seguro al sistema.
2. Gestión completa de clientes.
3. **Crear presupuestos con datos del vehículo y trabajos.**
4. **Enviar presupuestos por correo al cliente.**
5. **Aprobar presupuestos (crea automáticamente orden de trabajo).**
6. **Rechazar presupuestos.**
7. Asignar mecánicos a órdenes.
8. Cambiar estado de órdenes.
9. **Envío automático de correo al cliente (estado "Listo").**
10. Buscar y filtrar presupuestos y órdenes.

**Mecánico:**
1. Ver órdenes asignadas.
2. Actualizar estado de órdenes.
3. Agregar notas a órdenes.

## 6. Datos Básicos a Almacenar

### Cliente:
- ID
- Nombre completo
- Teléfono
- **Correo electrónico (obligatorio para notificaciones)**
- Fecha de registro

### Presupuesto:
- ID
- Número de presupuesto (correlativo)
- ID Cliente (referencia)
- **Datos del vehículo:**
  - Marca
  - Modelo
  - Año
  - Patente/Placa
  - Kilometraje
- Fecha de emisión
- Descripción del trabajo/problema
- Trabajos propuestos (texto descriptivo)
- Costo estimado (CLP)
- **Estado (Pendiente, Aprobado, Rechazado)**
- **Correo enviado (boolean)**
- **Fecha de envío de correo**
- **ID Orden de Trabajo generada (referencia, si fue aprobado)**
- Observaciones

### Orden de Trabajo:
- ID
- Número de orden (correlativo)
- **ID Presupuesto (referencia al presupuesto aprobado)**
- ID Cliente (referencia)
- ID Mecánico asignado (referencia)
- **Datos del vehículo (copiados del presupuesto):**
  - Marca
  - Modelo
  - Año
  - Patente/Placa
  - Kilometraje al ingreso
- Fecha de creación (cuando se aprobó el presupuesto)
- Fecha estimada de entrega
- Fecha de entrega real
- Descripción del trabajo (copiada del presupuesto)
- Trabajos realizados (puede ser editado/ampliado)
- Observaciones adicionales
- Estado (En Reparación, Listo, Entregado)
- Costo final (CLP)
- **Correo de "Listo" enviado (boolean)**
- **Fecha de envío de correo "Listo"**

### Mecánico:
- ID
- Nombre completo
- Teléfono
- Especialidad (opcional)
- Estado (Activo/Inactivo)

### Usuario (para autenticación):
- ID
- Username
- Password (encriptado)
- Rol (Administrador/Mecánico)
- ID Mecánico (referencia, si aplica)
- Estado (Activo/Inactivo)

## 7. Reglas del Negocio

### 7.1. Usuarios y Autenticación
- Solo usuarios registrados pueden acceder al sistema.
- Las contraseñas deben estar encriptadas con bcrypt.
- Los tokens JWT expiran en 24 horas.
- Solo el Administrador puede crear nuevos usuarios.

### 7.2. Clientes
- **El correo electrónico es obligatorio para enviar presupuestos y notificaciones.**
- No se puede eliminar un cliente que tenga presupuestos u órdenes asociadas.
- Los datos de contacto (teléfono, correo) deben ser válidos.

### 7.3. Presupuestos
- **Los estados posibles son: Pendiente, Aprobado, Rechazado.**
- Todo presupuesto nuevo se crea en estado "Pendiente".
- **Los datos del vehículo (marca, modelo, año, patente, kilometraje) son obligatorios.**
- La descripción del trabajo es obligatoria.
- El costo estimado debe ser mayor a cero.
- **Un presupuesto puede ser enviado por correo al cliente.**
- **Al aprobar un presupuesto, se crea automáticamente una orden de trabajo con los mismos datos.**
- **Un presupuesto aprobado NO puede ser modificado.**
- **Un presupuesto rechazado queda solo como registro histórico.**
- Solo el Administrador puede aprobar o rechazar presupuestos.

### 7.4. Órdenes de Trabajo
- **Toda orden de trabajo proviene de un presupuesto aprobado.**
- **Los estados posibles son: En Reparación → Listo → Entregado.**
- Al crear la orden (desde presupuesto aprobado), se crea en estado "En Reparación".
- **La orden mantiene referencia al ID del presupuesto origen.**
- Una orden debe tener asignado un mecánico.
- **Al cambiar estado a "Listo", el sistema debe enviar automáticamente un correo al cliente.**
- Los trabajos realizados pueden ser editados/ampliados durante la reparación.
- Solo el Administrador puede eliminar órdenes (solo si están en "En Reparación").
- No se pueden eliminar órdenes en estado "Listo" o "Entregado".

### 7.5. Asignación de Mecánicos
- Una orden debe tener un mecánico asignado.
- Un mecánico puede tener múltiples órdenes asignadas.
- Solo mecánicos activos pueden ser asignados.
- El Administrador puede reasignar órdenes.

### 7.6. Notificaciones por Correo
- **El correo de presupuesto se envía manualmente cuando el Administrador lo solicita.**
- **El correo de "Listo" se envía automáticamente cuando la orden cambia a ese estado.**
- Los correos deben incluir toda la información relevante.
- Se debe registrar la fecha y hora de envío.
- Si el envío falla, se deben realizar hasta 3 reintentos.
- El sistema debe validar que el cliente tenga un correo válido antes de intentar enviar.

### 7.7. Flujo Principal del Negocio
**Presupuesto → Aprobación → Orden de Trabajo → Notificación**
1. Se crea presupuesto con datos del vehículo y trabajos.
2. Se envía presupuesto por correo al cliente.
3. Cliente decide aprobar o rechazar.
4. Si aprueba: sistema crea orden automáticamente con los mismos datos.
5. Mecánico realiza trabajos.
6. Al terminar, se cambia estado a "Listo".
7. Sistema envía correo automático al cliente.
8. Cliente retira vehículo.
9. Se cambia estado a "Entregado".

### 7.8. Seguridad
- Todas las contraseñas se almacenan encriptadas.
- Los endpoints están protegidos con JWT.
- Los mecánicos solo acceden a sus órdenes asignadas.
- El Administrador tiene acceso total.

## 8. Requisitos Funcionales y No Funcionales

### Requisitos Funcionales:

**Gestión de Clientes:**
- Registrar, editar, eliminar y listar clientes.
- Buscar clientes por nombre o teléfono.
- Validar formato de correo electrónico.
- Ver historial de presupuestos y órdenes por cliente.

**Gestión de Presupuestos:**
- Crear presupuestos con datos de cliente y vehículo.
- Editar presupuestos en estado "Pendiente".
- Listar presupuestos con filtros por estado, cliente, fechas.
- **Enviar presupuesto por correo electrónico al cliente.**
- **Aprobar presupuesto (crea automáticamente orden de trabajo).**
- **Rechazar presupuesto.**
- Buscar presupuestos por número, cliente o patente.

**Gestión de Órdenes de Trabajo:**
- **Crear órdenes automáticamente al aprobar presupuesto.**
- Listar órdenes con filtros por estado, mecánico, cliente, fechas.
- Editar trabajos realizados y observaciones.
- Asignar/reasignar mecánico.
- Cambiar estado de órdenes (En Reparación → Listo → Entregado).
- Ver detalle completo de una orden.
- Buscar órdenes por número, patente o cliente.
- Ver referencia al presupuesto origen.

**Sistema de Notificaciones:**
- **Enviar manualmente presupuesto por correo al cliente.**
- **Enviar automáticamente correo al cliente cuando orden pasa a "Listo".**
- Incluir toda la información relevante en los correos.
- Registrar fecha de envío.
- Sistema de reintentos en caso de fallo.

**Gestión de Mecánicos:**
- Registrar, editar y listar mecánicos.
- Activar/desactivar mecánicos.
- Ver órdenes asignadas por mecánico.

**Autenticación:**
- Login con username y password.
- Generación de token JWT.
- Protección de endpoints por rol.
- Logout y expiración de sesión.

### Requisitos No Funcionales:

**Rendimiento:**
- Responder peticiones en menos de 2 segundos.
- Soportar al menos 100 clientes, 500 presupuestos y 500 órdenes.
- El correo debe enviarse en menos de 10 segundos.

**Seguridad:**
- Encriptación de contraseñas con bcrypt.
- Tokens JWT con expiración de 24 horas.
- Validación de datos en backend.
- Protección contra inyección NoSQL.

**Disponibilidad:**
- Sistema disponible 24/7.
- Correos con sistema de reintentos automáticos.

**Mantenibilidad:**
- Código documentado y estructurado.
- Uso de Docker para despliegue.
- README con instrucciones de instalación.
- API documentada (Postman/Swagger).

**Usabilidad:**
- Mensajes de error claros y descriptivos.
- Códigos HTTP apropiados.
- Respuestas en formato JSON consistente.

## 9. Definición del MVP

### Funcionalidades Incluidas:
- Autenticación JWT para Administrador y Mecánico.
- API REST completa con endpoints para:
  - CRUD de clientes con validación de correo.
  - **CRUD de presupuestos con aprobación/rechazo.**
  - **Envío manual de presupuestos por correo.**
  - **Creación automática de orden al aprobar presupuesto.**
  - Gestión de órdenes de trabajo.
  - Asignación de mecánicos.
  - Cambio de estados de órdenes.
  - **Envío automático de correo al cambiar a "Listo".**
  - Búsquedas y filtros.
- MongoDB para persistencia.
- Redis para cache de consultas frecuentes.
- Sistema de correo con Nodemailer.
- Proyecto dockerizado.
- Validaciones de todas las reglas de negocio.
- Documentación completa.

### Futuras Versiones:
- Interfaz web completa.
- Dashboard con gráficos y estadísticas.
- Sistema de inventario de repuestos.
- Historial detallado por vehículo.
- Notificaciones SMS/WhatsApp.
- Sistema de facturación.
- Reportes exportables (PDF, Excel).
- Firma digital de presupuestos/órdenes.

## 10. Lista Priorizada de Funcionalidades

### Prioridad Crítica (Imprescindible para MVP)
1. Autenticación JWT y gestión de usuarios.
2. CRUD de clientes con validación de correo.
3. **CRUD de presupuestos.**
4. **Envío manual de presupuestos por correo.**
5. **Aprobación de presupuestos (✓ crea orden automáticamente).**
6. **Rechazo de presupuestos.**
7. **Creación automática de orden desde presupuesto aprobado.**
8. Gestión de órdenes (editar, cambiar estado).
9. Asignación de mecánicos a órdenes.
10. **Envío automático de correo al estado "Listo".**
11. Persistencia en MongoDB.
12. Cache básico en Redis.
13. Proyecto dockerizado.
14. README con documentación.

### Prioridad Alta (Post-MVP inmediato)
1. Búsquedas y filtros avanzados.
2. Panel de mecánicos con sus órdenes.
3. Reportes básicos de presupuestos vs órdenes.
4. Plantillas personalizables de correo.

### Prioridad Media
1. Dashboard con estadísticas.
2. Exportación de datos.
3. Historial por vehículo.
4. Notificaciones adicionales.

### Prioridad Baja
1. Interfaz web completa.
2. Integración con WhatsApp.

## 11. Flujos Principales del Sistema

### 11.1. Autenticación
1. Usuario ingresa username y password.
2. Sistema valida credenciales.
3. Si son correctas, genera token JWT.
4. Token se incluye en el header de todas las peticiones protegidas.
5. Token expira en 24 horas.

### 11.2. Creación y Envío de Presupuesto
1. Cliente llega con su vehículo al taller.
2. Administrador verifica si el cliente existe.
3. Si no existe, lo registra con sus datos (incluyendo correo obligatorio).
4. Administrador crea nuevo presupuesto ingresando:
   - Selecciona o confirma cliente
   - Datos del vehículo (marca, modelo, año, patente, kilometraje)
   - Descripción del problema/trabajo solicitado
   - Trabajos propuestos
   - Costo estimado
5. Sistema guarda presupuesto en estado "Pendiente".
6. Administrador envía presupuesto por correo al cliente.
7. Sistema genera correo con formato profesional.
8. Sistema envía correo (reintenta hasta 3 veces si falla).
9. Sistema registra fecha de envío.
10. Cliente recibe presupuesto en su correo.

### 11.3. Aprobación de Presupuesto y Creación de Orden
1. Cliente revisa presupuesto y decide aprobarlo.
2. Cliente llama o llega al taller para confirmar.
3. Administrador busca el presupuesto en el sistema.
4. Administrador marca presupuesto como "Aprobado".
5. **Sistema automáticamente:**
   - Cambia estado del presupuesto a "Aprobado"
   - **Crea nueva orden de trabajo con los mismos datos:**
     - Copia ID de cliente
     - Copia datos del vehículo
     - Copia descripción del trabajo
     - Copia trabajos propuestos
     - Copia costo estimado
     - Establece estado inicial: "En Reparación"
     - Vincula presupuesto con orden (guarda ID presupuesto en orden)
   - Genera número de orden correlativo
6. Administrador asigna mecánico a la orden.
7. Orden queda lista para que el mecánico trabaje.

### 11.4. Rechazo de Presupuesto
1. Cliente decide no aprobar el presupuesto.
2. Administrador marca presupuesto como "Rechazado".
3. **Sistema NO crea orden de trabajo.**
4. Presupuesto queda como registro histórico.

### 11.5. Proceso de Reparación
1. Mecánico ve su orden asignada.
2. Realiza los trabajos especificados.
3. Puede agregar observaciones y detalles adicionales.
4. Si necesita agregar trabajos extra, edita "trabajos realizados".
5. Al terminar, actualiza estado a "Listo".
6. **Sistema detecta cambio a "Listo" y envía correo automático al cliente.**

### 11.6. Envío Automático de Correo "Listo"
1. Sistema detecta que orden cambió a estado "Listo".
2. Valida que el cliente tenga correo electrónico registrado.
3. Genera correo con formato profesional incluyendo:
   - Número de orden
   - Datos del vehículo (marca, modelo, patente)
   - Trabajos realizados
   - Fecha en que quedó listo
   - Costo final (si aplica)
   - Datos de contacto del taller
4. Intenta enviar correo.
5. Si falla, reintenta hasta 3 veces.
6. Registra fecha y resultado del envío en la orden.
7. Cliente recibe notificación en su correo.

### 11.7. Entrega del Vehículo
1. Cliente llega a retirar vehículo.
2. Administrador busca orden en estado "Listo".
3. Revisa con cliente los trabajos realizados.
4. Registra costo final.
5. Cambia estado a "Entregado".
6. Registra fecha de entrega real.

## 12. Modelo de Datos

### Relaciones:
- Cliente (1) → Presupuestos (N)
- Cliente (1) → Órdenes de Trabajo (N)
- **Presupuesto (1) → Orden de Trabajo (0..1)** [Relación uno a uno opcional]
- Mecánico (1) → Órdenes de Trabajo (N)
- Usuario (1) → Mecánico (0..1)

### Diagrama simplificado:
```
Cliente
  ├── Presupuestos
  │     └── Orden de Trabajo (si fue aprobado)
  └── Órdenes de Trabajo
          └── Mecánico asignado
```

### Flujo de datos:
```
Presupuesto (Pendiente) 
    ↓ [Aprobar]
Presupuesto (Aprobado) → Crea → Orden de Trabajo (En Reparación)
                                      ↓ [Termina trabajo]
                                 Orden (Listo) → Envía correo
                                      ↓ [Cliente retira]
                                 Orden (Entregado)
```
## 13. Consideraciones Técnicas

### Stack Tecnológico:
- Backend: Node.js con Express
- Base de Datos: MongoDB con Mongoose
- Cache: Redis
- Autenticación: JWT con bcrypt
- Correo: Nodemailer
- Containerización: Docker y docker-compose

### Endpoints Principales:

**Autenticación:**
- POST /api/auth/login
- POST /api/auth/logout
- GET /api/auth/me

**Clientes:**
- GET /api/clients
- GET /api/clients/:id
- POST /api/clients
- PUT /api/clients/:id
- DELETE /api/clients/:id
- GET /api/clients/search?q=
- GET /api/clients/:id/quotes (presupuestos del cliente)
- GET /api/clients/:id/orders (órdenes del cliente)

**Presupuestos:**
- GET /api/quotes
- GET /api/quotes/:id
- POST /api/quotes
- PUT /api/quotes/:id (solo si está Pendiente)
- DELETE /api/quotes/:id
- POST /api/quotes/:id/send-email (enviar por correo)
- PUT /api/quotes/:id/approve (aprobar → crea orden automáticamente)
- PUT /api/quotes/:id/reject (rechazar)
- GET /api/quotes?status=&client=&date=

**Órdenes de Trabajo:**
- GET /api/orders
- GET /api/orders/:id
- PUT /api/orders/:id (editar trabajos/observaciones)
- PUT /api/orders/:id/status (cambiar estado)
- PUT /api/orders/:id/assign (asignar mecánico)
- DELETE /api/orders/:id (solo si está En Reparación)
- GET /api/orders?status=&mechanic=&client=

**Mecánicos:**
- GET /api/mechanics
- GET /api/mechanics/:id
- POST /api/mechanics
- PUT /api/mechanics/:id
- GET /api/mechanics/:id/orders

## 14. Cronograma Sugerido (4 semanas)

### Semana 1: Fundamentos
- Configuración inicial del proyecto.
- Setup de Docker y docker-compose.
- Conexión a MongoDB y Redis.
- Sistema de autenticación JWT.
- CRUD de clientes con validaciones.
- Configuración de sistema de correo (Nodemailer).

### Semana 2: Presupuestos
- CRUD completo de presupuestos.
- Validaciones de datos del vehículo.
- **Envío manual de presupuestos por correo.**
- **Lógica de aprobación de presupuestos.**
- **Lógica de rechazo de presupuestos.**
- Búsquedas y filtros de presupuestos.

### Semana 3: Órdenes y Mecánicos
- **Creación automática de orden al aprobar presupuesto.**
- **Vinculación presupuesto ↔ orden.**
- Gestión de órdenes (editar, estados).
- CRUD de mecánicos.
- Asignación de mecánicos a órdenes.
- **Envío automático de correo al estado "Listo".**

### Semana 4: Refinamiento y Documentación
- Cache con Redis.
- Validaciones completas de reglas de negocio.
- Pruebas de integración.
- **Pruebas exhaustivas del flujo presupuesto → orden.**
- **Pruebas de envío de correos (manual y automático).**
- Documentación completa (API, instalación, uso).
- Ajustes finales y optimización.