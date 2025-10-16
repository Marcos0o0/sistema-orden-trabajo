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