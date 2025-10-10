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
