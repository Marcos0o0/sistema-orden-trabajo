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
