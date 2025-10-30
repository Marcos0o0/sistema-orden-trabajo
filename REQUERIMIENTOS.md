# REQUERIMIENTOS DEL SISTEMA

## 1. Descripción del Cliente y Problema Principal

**Cliente:** Taller mecánico automotriz.

**Problema Principal:**
El taller mecánico maneja todas las órdenes de trabajo de forma manual en cuadernos y papeles sueltos, lo que genera pérdida de información y desorganización. No hay forma de enviar documentación formal a los clientes por correo electrónico, obligándolos a llamar constantemente para conocer el estado de sus vehículos.

**Necesidad:**
Crear un sistema digital que permita gestionar presupuestos y órdenes de trabajo de manera eficiente, con notificación automática por correo electrónico al cliente, mejorando la comunicación y profesionalizando el servicio.

## 2. Plazo y Alcance del Proyecto

### Plazo de Desarrollo:
4 semanas para completar el MVP funcional.

### Alcance del Proyecto:

**Incluido en el MVP:**
- Sistema de autenticación con JWT.
- Gestión completa de clientes (crear, editar, eliminar, listar, buscar).
- Gestión de presupuestos (crear, editar, enviar por correo, aprobar/rechazar).
- Creación automática de orden de trabajo al aprobar presupuesto.
- Gestión de órdenes de trabajo (listar, editar, cambiar estado).
- Asignación de mecánicos a órdenes de trabajo.
- Estados de órdenes: En Reparación, Listo, Entregado.
- Envío automático de correo electrónico al cliente cuando la orden cambia a estado "Listo".
- Base de datos MongoDB.
- Cache con Redis para consultas frecuentes.
- Proyecto dockerizado con documentación completa.

**Excluido del MVP (versiones futuras):**
- Interfaz gráfica web completa (solo API REST).
- Gestión de inventario y repuestos.
- Roles adicionales (Cliente con login, Recepcionista).
- Reportes y dashboards con gráficos.
- Sistema de facturación.
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

## 4. Permisos y funciones indispensables por perfil

### Administrador
- Acceso completo al sistema.  
- Autenticación y control de acceso.  
- Gestión de clientes (crear, editar, eliminar).  
- Gestión de presupuestos:
  - Crear, editar, enviar, aprobar y rechazar presupuestos.  
  - Envío de presupuestos y notificaciones por correo.  
- Gestión de órdenes de trabajo:
  - Crear automáticamente desde presupuestos aprobados.  
  - Asignar mecánicos.  
  - Cambiar estados (en progreso, listo, entregado, etc.).  
- Buscar y filtrar presupuestos u órdenes.  
- Ver historial completo de presupuestos y órdenes.  

### Mecánico
- Ver órdenes asignadas.  
- Actualizar estado de sus órdenes.  
- Agregar notas y observaciones.  

## 6. Datos Básicos a Almacenar

### Cliente:
- ID
- Nombre
- Apellido parterno
- Apellido materno
- Teléfono
- Correo electrónico
- Fecha de registro

### Presupuesto:
- ID
- Número de presupuesto (correlativo)
- ID Cliente (referencia)
- Datos del vehículo:
  - Marca
  - Modelo
  - Año
  - Patente/Placa
  - Kilometraje
- Fecha de emisión
- Descripción del trabajo/problema
- Trabajos propuestos (texto descriptivo)
- Costo estimado (CLP)
- Estado (Pendiente, Aprobado, Rechazado)
- Token de aprobación/rechazo (para links en correo)
- Fecha de expiración del token
- Token usado (true o false)
- Correo enviado (true o false)
- Fecha de envío de correo
- ID Orden de Trabajo generada (referencia, si fue aprobado)
- Observaciones

### Orden de Trabajo:
- ID
- Número de orden (correlativo)
- ID Presupuesto (referencia al presupuesto aprobado)
- ID Mecánico asignado (referencia)
- Fecha de creación (cuando se aprobó el presupuesto)
- Fecha estimada de entrega
- Fecha de entrega real
- Observaciones adicionales (solo de la orden)
- Estado (En Reparación, Listo, Entregado)
- Costo final (puede diferir del estimado)
- Correo de "Listo" enviado (true o false)
- Fecha de envío de correo "Listo"

Nota: La orden NO duplica datos del vehículo, cliente ni trabajos. 
Todo eso se obtiene consultando el presupuesto vinculado.

### Mecánico:
- ID
- Nombre
- Apellido Paterno
- Teléfono

## 7. Reglas del Negocio

### 7.1. Usuarios y Autenticación
- Solo usuarios registrados pueden acceder al sistema.
- Las contraseñas deben estar encriptadas con bcrypt.
- Los tokens JWT expiran en 24 horas.
- Solo el Administrador puede crear nuevos usuarios.

### 7.2. Clientes
- El correo electrónico es obligatorio para enviar presupuestos y notificaciones.
- No se puede eliminar un cliente que tenga presupuestos u órdenes asociadas.
- Los datos de contacto (teléfono, correo) deben ser válidos.

### 7.3. Presupuestos
- Los estados posibles son: Pendiente, Aprobado, Rechazado.
- Todo presupuesto nuevo se crea en estado "Pendiente".
- Los datos del vehículo (marca, modelo, año, patente, kilometraje) son obligatorios.
- La descripción del trabajo es obligatoria.
- El costo estimado debe ser mayor a cero.
- Un presupuesto puede ser enviado por correo al cliente.
- Al aprobar un presupuesto, se crea automáticamente una orden de trabajo vinculada mediante referencia al ID del presupuesto.
- Un presupuesto aprobado NO puede ser modificado.
- Un presupuesto rechazado queda solo como registro histórico.
- Solo el Administrador puede aprobar o rechazar presupuestos.

### 7.4. Órdenes de Trabajo
- Toda orden de trabajo proviene de un presupuesto aprobado.
- Los estados posibles son: En Reparación → Listo → Entregado.
- Al crear la orden (desde presupuesto aprobado), se crea en estado "En Reparación".
- La orden mantiene referencia al ID del presupuesto origen.
- Los datos del cliente, vehículo y trabajos originales se obtienen consultando el presupuesto vinculado.
- Una orden debe tener asignado un mecánico.
- Al cambiar estado a "Listo", el sistema debe enviar automáticamente un correo al cliente.
- Las observaciones y trabajos adicionales pueden ser agregados durante la reparación.
- Solo el Administrador puede eliminar órdenes (solo si están en "En Reparación").
- No se pueden eliminar órdenes en estado "Listo" o "Entregado".

### 7.5. Asignación de Mecánicos
- Una orden debe tener un mecánico asignado.
- Un mecánico puede tener múltiples órdenes asignadas.
- El Administrador puede reasignar órdenes.

### 7.6. Notificaciones por Correo
- El correo de presupuesto se envía manualmente cuando el Administrador lo solicita.
- El correo de "Listo" se envía automáticamente cuando la orden cambia a ese estado.
- Los correos deben incluir toda la información relevante.
- Se debe registrar la fecha y hora de envío.
- Si el envío falla, se deben realizar hasta 3 reintentos.
- El sistema debe validar que el cliente tenga un correo válido antes de intentar enviar.

### 7.7. Flujo Principal del Negocio
**Presupuesto → Aprobación → Orden de Trabajo → Notificación**
1. Se crea presupuesto con datos del vehículo y trabajos.
2. Se envía presupuesto por correo al cliente con links de aprobación/rechazo.
3. Cliente decide aprobar o rechazar mediante click en link.
4. Si aprueba: sistema crea orden automáticamente vinculada al presupuesto mediante referencia.
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
- Enviar presupuesto por correo electrónico al cliente.
- Aprobar presupuesto (crea automáticamente orden de trabajo).
- Rechazar presupuesto.
- Buscar presupuestos por número, cliente o patente.

**Gestión de Órdenes de Trabajo:**
- Crear órdenes automáticamente al aprobar presupuesto (solo referencia al presupuesto).
- Listar órdenes con filtros por estado, mecánico, cliente, fechas.
- Editar observaciones adicionales de la orden.
- Asignar/reasignar mecánico.
- Cambiar estado de órdenes (En Reparación → Listo → Entregado).
- Ver detalle completo incluyendo datos del presupuesto vinculado.
- Buscar órdenes por número, patente o cliente.

**Sistema de Notificaciones:**
- Enviar manualmente presupuesto por correo al cliente.
- Enviar automáticamente correo al cliente cuando orden pasa a "Listo".
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
  - CRUD de presupuestos con aprobación/rechazo.
  - Envío manual de presupuestos por correo.
  - Creación automática de orden al aprobar presupuesto.
  - Gestión de órdenes de trabajo.
  - Asignación de mecánicos.
  - Cambio de estados de órdenes.
  - Envío automático de correo al cambiar a "Listo".
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
- Sistema de facturación.
- Reportes exportables (PDF, Excel).
- Firma digital de presupuestos/órdenes.

## 10. Lista Priorizada de Funcionalidades

### Prioridad Crítica (Imprescindible para MVP)
1. Autenticación JWT y gestión de usuarios.
2. CRUD de clientes con validación de correo.
3. CRUD de presupuestos.
4. Envío manual de presupuestos por correo.
5. Aprobación de presupuestos (✓ crea orden automáticamente).
6. Rechazo de presupuestos.
7. Creación automática de orden desde presupuesto aprobado.
8. Gestión de órdenes (editar, cambiar estado).
9. Asignación de mecánicos a órdenes.
10. Envío automático de correo al estado "Listo".
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

## 11. Flujos Principales del Sistema

### 11.1. Autenticación
1. Usuario ingresa username y password.
2. Sistema valida credenciales.
3. Si son correctas, genera token JWT.
4. Token se incluye en el header de todas las peticiones protegidas.
5. Token expira en 24 horas.

### 11.2. Creación y Envío de Presupuesto
1. Cliente llega con su vehículo al taller.
2. Administrador verifica si el cliente existe, si no, lo registra con correo obligatorio.
3. Administrador crea presupuesto ingresando:
   - Cliente
   - Datos del vehículo (marca, modelo, año, patente, kilometraje)
   - Descripción del problema
   - Trabajos propuestos
   - Costo estimado
4. Sistema guarda presupuesto en estado "Pendiente".
5. Sistema genera token único de seguridad para ese presupuesto.
6. Administrador envía presupuesto por correo al cliente.
7. Sistema genera correo profesional con:
   - Detalles del presupuesto
   - Link "Aprobar": `https://ejemplo.com/api/quotes/:id/approve?token=:token`
   - Link "Rechazar": `https://ejemplo.com/api/quotes/:id/reject?token=:token`
8. Sistema envía correo (reintenta hasta 3 veces si falla).
9. Sistema registra fecha de envío.

### 11.3. Aprobación de Presupuesto y Creación Automática de Orden
1. Cliente hace click en "Aprobar Presupuesto" del correo.
2. Sistema valida token de seguridad.
3. Sistema cambia estado del presupuesto a "Aprobado".
4. Sistema crea automáticamente orden de trabajo:
   - Referencia al ID del presupuesto (única fuente de datos)
   - Estado inicial: "En Reparación"
   - Número de orden correlativo
5. Sistema vincula orden con presupuesto.
6. Sistema muestra página de confirmación al cliente.
7. Sistema envía correo de confirmación con número de orden.
8. Sistema notifica al administrador.
9. Administrador asigna mecánico a la orden.

### 11.4. Rechazo de Presupuesto
1. Cliente hace click en "Rechazar Presupuesto" del correo.
2. Sistema valida token de seguridad.
3. Sistema cambia estado del presupuesto a "Rechazado".
4. Sistema muestra página de confirmación.
5. No se crea orden de trabajo.
6. Presupuesto queda como registro histórico.

### 11.5. Proceso de Reparación
1. Mecánico consulta sus órdenes asignadas (estado: "En Reparación").
2. Mecánico realiza trabajos especificados.
3. Mecánico puede agregar observaciones y trabajos adicionales.
4. Al terminar, mecánico cambia estado a "Listo".
5. Sistema ejecuta automáticamente notificación al cliente.

### 11.6. Notificación Automática de Vehículo Listo
Trigger: Orden cambia a estado "Listo"

1. Sistema detecta cambio de estado.
2. Sistema valida que cliente tenga correo registrado.
3. Sistema genera correo profesional con detalles de la orden y vehículo listo para retiro.
4. Sistema envía correo (hasta 3 reintentos si falla).
5. Sistema registra fecha y resultado del envío.

### 11.7. Entrega del Vehículo
1. Cliente llega al taller.
2. Administrador busca orden en estado "Listo".
3. Administrador revisa trabajos realizados y costo con cliente.
4. Cliente realiza pago.
5. Administrador cambia estado a "Entregado".
6. Sistema registra fecha de entrega.

## 12. Modelo de Datos

### Relaciones:
- Cliente (1) → Presupuestos (N)
- Cliente (1) → Órdenes de Trabajo (N)
- Presupuesto (1) → Orden de Trabajo (0..1) [Relación uno a uno opcional]
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
- GET /api/quotes/:id/approve?token={token} (aprobación por cliente desde correo)
- PUT /api/quotes/:id/approve (aprobación manual por administrador)
- GET /api/quotes/:id/reject?token={token} (rechazo por cliente desde correo)
- PUT /api/quotes/:id/reject (rechazo manual por administrador)
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
- Envío manual de presupuestos por correo.
- Lógica de aprobación de presupuestos.
- Lógica de rechazo de presupuestos.
- Búsquedas y filtros de presupuestos.

### Semana 3: Órdenes y Mecánicos
- Creación automática de orden al aprobar presupuesto.
- Vinculación presupuesto ↔ orden.
- Gestión de órdenes (editar, estados).
- CRUD de mecánicos.
- Asignación de mecánicos a órdenes.
- Envío automático de correo al estado "Listo".

### Semana 4: Refinamiento y Documentación
- Cache con Redis.
- Validaciones completas de reglas de negocio.
- Pruebas de integración.
- Pruebas exhaustivas del flujo presupuesto → orden.
- Pruebas de envío de correos (manual y automático).
- Documentación completa (API, instalación, uso).
- Ajustes finales y optimización.

## 15. Checklist del Proyecto

**Configuración Inicial:**
- [ ] Grupo formado (3-4 integrantes).
- [ ] Repositorio en GitHub/GitLab.
- [ ] Gitflow configurado (main y develop).
- [ ] REQUERIMIENTOS.md completo.
- [ ] README.md con instrucciones.

**Infraestructura:**
- [ ] Docker y docker-compose funcionando.
- [ ] MongoDB conectado.
- [ ] Redis configurado.
- [ ] Nodemailer configurado para envío de correos.

**Autenticación:**
- [ ] Login funcional con JWT.
- [ ] Middleware de autenticación.
- [ ] Encriptación de contraseñas.
- [ ] Roles (Administrador/Mecánico).

**Gestión de Clientes:**
- [ ] CRUD completo.
- [ ] Validación de correo electrónico obligatorio.
- [ ] Búsqueda de clientes.
- [ ] Ver presupuestos y órdenes por cliente.
- [ ] No eliminar clientes con presupuestos/órdenes.

**Gestión de Presupuestos (Importante):**
- [ ] Crear presupuestos con datos de cliente y vehículo.
- [ ] Estados: Pendiente, Aprobado, Rechazado.
- [ ] Editar presupuestos solo en estado Pendiente.
- [ ] Validación de datos del vehículo (marca, modelo, año, patente, km).
- [ ] Envío manual de presupuesto por correo.
- [ ] Formato profesional del correo de presupuesto.
- [ ] Aprobar presupuesto (botón/endpoint).
- [ ] Rechazar presupuesto.
- [ ] Listar con filtros (estado, cliente, fecha).
- [ ] Buscar presupuestos.

**Creación Automática de Orden (Importante):**
- [ ] Al aprobar presupuesto, crear orden automáticamente.
- [ ] Referenciar presupuesto en la orden mediante ID (NO copiar datos).
- [ ] Vincular orden con presupuesto (guardar ID de orden en presupuesto).
- [ ] Establecer estado inicial "En Reparación".
- [ ] Generar número de orden correlativo.
- [ ] Presupuesto aprobado no puede modificarse.

**Gestión de Órdenes:**
- [ ] Listar órdenes.
- [ ] Ver detalle de orden.
- [ ] Editar trabajos realizados y observaciones.
- [ ] Estados: En Reparación, Listo, Entregado.
- [ ] Cambiar estado respetando flujo.
- [ ] Asignar/reasignar mecánico.
- [ ] Buscar órdenes.
- [ ] Filtrar por estado, mecánico, cliente.

**Sistema de Correo (Importante):**
- [ ] Configuración correcta de SMTP.
- [ ] Envío manual de presupuesto funcional.
- [ ] Envío automático al cambiar orden a "Listo".
- [ ] Formato profesional de ambos correos.
- [ ] Sistema de reintentos (3 intentos).
- [ ] Registro de fecha de envío.
- [ ] Validación de correo del cliente.
- [ ] Logs de envíos (éxito/fallo).

**Gestión de Mecánicos:**
- [ ] CRUD de mecánicos.
- [ ] Ver órdenes asignadas por mecánico.
- [ ] Estados activo/inactivo.

**Validaciones:**
- [ ] Correo obligatorio en clientes.
- [ ] Datos de vehículo obligatorios en presupuestos.
- [ ] Flujo de estados correcto.
- [ ] Presupuesto aprobado no modificable.
- [ ] Solo un mecánico por orden.
- [ ] No eliminar con dependencias.

**Documentación:**
- [ ] API documentada (Postman/Swagger).
- [ ] Flujo presupuesto → orden documentado.
- [ ] README completo.
- [ ] Instrucciones de instalación.
- [ ] Ejemplos de uso de endpoints clave.

**Pruebas:**
- [ ] Autenticación.
- [ ] CRUD de cada módulo.
- [ ] Flujo completo: crear presupuesto → enviar → aprobar → orden creada.
- [ ] Envío de correo de presupuesto.
- [ ] Envío automático de correo "Listo".
- [ ] Cambios de estado.
- [ ] Validaciones de negocio.

**Entrega Final:**
- [ ] Proyecto funciona con docker-compose.
- [ ] Todos los endpoints funcionando.
- [ ] Flujo presupuesto → orden funcionando perfectamente.
- [ ] Correos enviándose correctamente.
- [ ] Base de datos con datos de prueba.
- [ ] Código limpio y comentado.
- [ ] Al menos 10 commits descriptivos en develop.

## 16. Casos de Uso Principales

### Caso 1: Creación y Envío de Presupuesto
**Actores:** Administrador, Cliente

**Flujo Principal:**
1. Cliente llega al taller con problema en su vehículo.
2. Administrador verifica si cliente existe, si no, lo registra con email obligatorio.
3. Administrador crea presupuesto ingresando:
   - Cliente
   - Datos del vehículo (marca, modelo, año, patente, km)
   - Descripción del problema
   - Trabajos propuestos
   - Costo estimado
4. Sistema guarda presupuesto en estado "Pendiente".
5. Administrador solicita enviar presupuesto por correo.
6. Sistema valida que cliente tenga email.
7. Sistema genera correo profesional con toda la información.
8. Sistema envía correo (con reintentos si falla).
9. Sistema registra fecha de envío.
10. Cliente recibe presupuesto en su correo y lo revisa.

### Caso 2: Cliente Aprueba Presupuesto → Creación Automática de Orden
**Actores:** Cliente, Sistema, Administrador

**Flujo Principal:**
1. Cliente recibe presupuesto por correo con links de aprobación/rechazo con tokens de seguridad.
2. Cliente hace click en "Aprobar Presupuesto".
3. Sistema valida token de seguridad y verifica que no haya expirado.
4. Sistema automáticamente:
   - Cambia estado del presupuesto a "Aprobado"
   - Crea nueva orden de trabajo:
     - Referencia al ID del presupuesto (única fuente de datos del vehículo y trabajos)
     - Estado inicial: "En Reparación"
     - Número de orden correlativo
   - Vincula presupuesto con orden (guarda ID de orden en presupuesto)
   - Invalida token para evitar aprobaciones duplicadas
5. Sistema muestra página de confirmación: "Presupuesto aprobado. Orden #00123 creada exitosamente."
6. Sistema envía correo de confirmación al cliente con número de orden.
7. Sistema notifica al administrador sobre nueva orden pendiente de asignación.
8. Administrador asigna mecánico a la orden.
9. Presupuesto aprobado queda bloqueado (no se puede modificar).

**Flujo Alternativo - Cliente Rechaza:**
1. Cliente hace click en "Rechazar Presupuesto" del correo.
2. Sistema valida token de seguridad.
3. Sistema cambia estado del presupuesto a "Rechazado".
4. Sistema muestra página de confirmación: "Presupuesto rechazado. Gracias por su tiempo."
5. Sistema NO crea orden de trabajo.
6. Presupuesto queda solo como registro histórico.

**Flujo Alternativo - Aprobación Manual por Administrador:**
1. Cliente llama al taller confirmando aprobación verbalmente.
2. Administrador busca presupuesto en sistema.
3. Administrador usa endpoint PUT /api/quotes/:id/approve (con JWT).
4. Sistema ejecuta el mismo proceso automático del punto 4 del flujo principal.

### Caso 3: Reparación y Notificación Automática
**Actores:** Mecánico, Sistema, Cliente

**Flujo Principal:**
1. Mecánico trabaja en la orden asignada (estado "En Reparación").
2. Realiza los trabajos especificados.
3. Agrega observaciones o trabajos adicionales si es necesario.
4. Al terminar, cambia estado de orden a "Listo".
5. Sistema detecta cambio a "Listo".
6. Sistema ejecuta notificación automática (ver flujo 11.6).

### Caso 4: Entrega del Vehículo
**Actores:** Cliente, Administrador

**Flujo Principal:**
1. Cliente llega al taller a retirar vehículo.
2. Administrador busca orden en estado "Listo".
3. Revisa trabajos realizados con el cliente.
4. Confirma costo final.
5. Cliente realiza el pago.
6. Administrador cambia estado a "Entregado".
7. Registra fecha de entrega real.
8. Orden completada forma parte del historial del cliente.

### Caso 5: Cliente Frecuente con Nuevo Problema
**Actores:** Cliente, Administrador

**Flujo Principal:**
1. Cliente que ya tiene historial regresa con nuevo problema.
2. Administrador busca cliente existente.
3. Sistema muestra presupuestos y órdenes anteriores.
4. Administrador puede ver trabajos previos del mismo vehículo.
5. Crea nuevo presupuesto basándose en el contexto.
6. Sigue el flujo normal de presupuesto → aprobación → orden.

## 17. Métricas de Éxito del Sistema

### Métricas Funcionales:
- **Tasa de conversión presupuesto → orden:** Al menos 60% de presupuestos aprobados.
- **Correos de presupuesto entregados:** 95% o más exitosos.
- **Correos de "Listo" entregados:** 95% o más exitosos.
- **Tiempo de creación de presupuesto:** Menos de 5 minutos.
- **Creación automática de orden:** 100% exitosa al aprobar.

### Métricas Técnicas:
- **Tiempo de respuesta API:** < 2 segundos.
- **Disponibilidad del sistema:** 99% uptime.
- **Tasa de error en envío de correos:** < 5%.
- **Tiempo de creación automática de orden:** < 1 segundo.

### Métricas de Usabilidad:
- **Claridad de correos:** Información completa y legible.
- **Facilidad de aprobación:** 1 click para aprobar presupuesto.
- **Trazabilidad:** 100% de órdenes vinculadas a su presupuesto origen.

## 18. Riesgos y Mitigaciones

### Riesgos Técnicos:

**Riesgo 1: Fallo en creación automática de orden**
- **Impacto:** Crítico - Presupuesto aprobado pero sin orden.
- **Mitigación:** 
  - Usar transacciones de BD (todo o nada)
  - Logs detallados del proceso
  - Validación previa de datos
  - Sistema de rollback si falla

**Riesgo 2: Fallo en envío de correos**
- **Impacto:** Alto - Cliente no recibe información.
- **Mitigación:** 
  - Sistema de 3 reintentos automáticos
  - Logs detallados de intentos
  - Validación de configuración SMTP
  - Cola de reintentos con Redis

**Riesgo 3: Desvinculación presupuesto-orden**
- **Impacto:** Medio - Pérdida de trazabilidad.
- **Mitigación:** 
  - Campo obligatorio de referencia
  - Validación al crear orden
  - Índices en BD para búsqueda rápida

**Riesgo 4: Modificación de presupuesto aprobado**
- **Impacto:** Alto - Inconsistencia con orden creada.
- **Mitigación:** 
  - Bloqueo automático al aprobar
  - Validación en endpoints de edición
  - Auditoría de cambios

### Riesgos de Negocio:

**Riesgo 5: Cliente no recibe correo (spam, email inválido)**
- **Impacto:** Alto - Fallo en comunicación.
- **Mitigación:** 
  - Validación estricta de formato de email
  - Confirmación de envío en interfaz
  - Registro de todos los intentos
  - Opción de reenvío manual

**Riesgo 6: Pérdida de referencia presupuesto-orden**
- **Impacto:** Alto - Orden sin acceso a datos del vehículo.
- **Mitigación:** 
  - Validación exhaustiva de referencia al crear orden
  - Índices en BD para consultas rápidas
  - Validación de integridad referencial
  - No permitir eliminar presupuestos con órdenes asociadas

**Riesgo 7: Pérdida de datos en el proceso**
- **Impacto:** Crítico - Pérdida de presupuestos u órdenes.
- **Mitigación:** 
  - Backups automáticos diarios
  - Validaciones de integridad
  - Transacciones atómicas
  - Logs de auditoría

## 19. Glosario de Términos

- **Presupuesto:** Cotización previa que detalla trabajos y costos antes de iniciar una reparación. Puede ser aprobado o rechazado por el cliente.
- **Orden de Trabajo:** Documento que autoriza y registra una reparación en curso. Se crea automáticamente al aprobar un presupuesto.
- **Aprobar:** Acción de aceptar un presupuesto, lo que genera automáticamente una orden de trabajo vinculada.
- **Rechazar:** Acción de declinar un presupuesto. No genera orden de trabajo.
- **Estado (Presupuesto):** Puede ser Pendiente, Aprobado o Rechazado.
- **Estado (Orden):** Puede ser En Reparación, Listo o Entregado.
- **Vinculación:** Relación entre presupuesto aprobado y orden creada, permite trazabilidad.
- **Notificación Automática:** Correo enviado sin intervención manual cuando la orden cambia a "Listo".
- **Mecánico:** Técnico asignado responsable de realizar la reparación.

## 20. Notas Finales para el Desarrollo

### Consideraciones Críticas:

1. **Flujo Presupuesto → Orden es EL CORE del sistema:**
   - Este es el flujo más importante del negocio
   - Debe ser robusto y sin errores
   - Usar transacciones para garantizar consistencia

2. **Creación Automática de Orden:**
   - Al aprobar presupuesto, la orden se crea SIN intervención manual
   - La orden SOLO guarda referencia al ID del presupuesto (NO copia datos)
   - Los datos del vehículo, cliente y trabajos se obtienen consultando el presupuesto vinculado
   - Establecer siempre estado inicial "En Reparación"
   - Vincular siempre con el presupuesto origen (bidireccional)

3. **Vinculación Presupuesto-Orden:**
   - Implementar campo `quoteId` en la orden
   - Implementar campo `orderId` en el presupuesto (cuando se apruebe)
   - Permite navegación bidireccional

4. **Dos Tipos de Correos:**
   - **Manual:** Presupuesto (enviado por solicitud del admin)
   - **Automático:** Notificación "Listo" (trigger al cambiar estado)

5. **Bloqueo de Presupuesto Aprobado:**
   - Una vez aprobado, NO se puede editar
   - Evita inconsistencias con la orden creada
   - Implementar validación en endpoints de edición

6. **Estados Claros:**
   - Presupuesto: Pendiente → Aprobado/Rechazado
   - Orden: En Reparación → Listo → Entregado
   - No permitir saltos de estado

7. **Validaciones Estrictas:**
   - Email obligatorio en clientes (crítico para correos)
   - Datos de vehículo obligatorios
   - Costo estimado > 0
   - Estados válidos

8. **Transacciones:**
   - Al aprobar presupuesto: todo o nada
   - Si falla creación de orden, revertir cambio de estado
   - Logs detallados para debugging

9. **Sistema de Correos Robusto:**
   - Reintentos automáticos (3 intentos)
   - Timeout apropiado
   - Manejo de errores claro
   - Logs de todos los intentos

10. **Testing Exhaustivo:**
    - Probar flujo completo múltiples veces
    - Casos de éxito y fallo
    - Validar que la referencia al presupuesto funciona correctamente
    - Validar que los datos del vehículo se obtienen del presupuesto vinculado
    - Validar que correos se envían
    - Probar aprobación por link con token
    - Probar tokens expirados e inválidos
    - Probar aprobación manual por administrador

### Secuencia de Implementación:

**Fase 1 - Base (Semana 1):**
1. Setup proyecto, Docker, MongoDB, Redis
2. Autenticación JWT
3. CRUD Clientes con validación email
4. Configuración Nodemailer

**Fase 2 - Presupuestos (Semana 2):**
1. Modelo y CRUD de Presupuestos
2. Validaciones de negocio
3. Envío manual de presupuesto por correo
4. Estados de presupuesto
5. Búsquedas y filtros

**Fase 3 - Órdenes y Automatización (Semana 3):**
1. Modelo de Órdenes
2. Lógica de aprobación de presupuesto
3. Creación automática de orden al aprobar
4. Vinculación presupuesto ↔ orden
5. CRUD Mecánicos
6. Asignación de mecánicos
7. Envío automático de correo al estado "Listo"

**Fase 4 - Refinamiento (Semana 4):**
1. Validaciones completas
2. Manejo de errores robusto
3. Cache con Redis
4. Pruebas exhaustivas
5. Documentación completa
6. Optimización

---

**Última actualización:** Octubre 2025  
**Versión:** 1.0  
**Estado:** Listo para desarrollo  
**Equipo:** [Marcos Godoy, Alvaro Sandoval, Vicente Ortiz y Martin Valdebenito]