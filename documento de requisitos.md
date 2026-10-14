# Documento de Requisitos: Sistema de Facturación y Cobros (TurbineH)

## Requisitos de negocio

Los requisitos de negocio describen los objetivos generales del proyecto interno para la startup TurbineH:

- Simplificar la gestión administrativa interna de clientes, facturas y cobros para que el equipo pueda centrarse en su producto principal.
- Automatizar el seguimiento de cobros y los recordatorios de impago para optimizar el flujo de caja.
- Proveer una operativa rápida y multidispositivo que permita la gestión en remoto.
- Garantizar la escalabilidad administrativa del sistema para soportar un volumen futuro de miles de clientes.

## Requisitos de usuario

Los requisitos de usuario describen las necesidades y servicios que el sistema debe ofrecer según los distintos perfiles de la empresa:

- El administrador podrá gestionar el alta de información, la generación de facturas y el seguimiento general de los cobros de todos los clientes.
- El usuario con rol de "lector" podrá consultar, editar y descargar únicamente las facturas y datos de los clientes que se le hayan asignado específicamente.
- El superadministrador podrá realizar configuraciones profundas del sistema (como cambiar los días estándar de vencimiento de las facturas) y recibir copias de seguridad del sistema.
- El administrador financiero podrá visualizar un panel de control (dashboard) con el estado de la facturación semanal, mensual y trimestral.

## Requisitos funcionales

### Gestión de Accesos y Usuarios

- **FR1.** El sistema requerirá inicio de sesión mediante correo electrónico y contraseña.
- **FR2.** El sistema validará que el correo electrónico introducido para el login pertenezca exclusivamente al dominio corporativo de TurbineH.
  - Dependencias: FR1.
- **FR3.** El sistema gestionará tres roles de acceso diferenciados: Superadministrador, Administrador y Lector.
- **FR4.** El sistema permitirá al rol Administrador y Superadministrador asignar clientes específicos a los usuarios con rol Lector.

### Gestión de Clientes

- **FR5.** El sistema permitirá importar la base de datos inicial de clientes (personas y empresas) mediante la carga de un archivo CSV o integración directa con el CRM.
- **FR6.** El sistema impedirá la baja definitiva de un cliente si este tiene facturas pendientes de pago, requiriendo que se complete el flujo de cobro antes de procesar la baja.
- **FR7.** El sistema permitirá filtrar el listado de clientes por ubicación y por temática de trabajo.

### Gestión de Facturación

- **FR8.** El sistema permitirá la creación de facturas recurrentes (mensuales) y facturas puntuales.
- **FR9.** El sistema generará automáticamente un documento PDF para cada factura que respete la estética corporativa, incluyendo numeración correlativa y datos fiscales.
- **FR10.** El sistema enviará automáticamente las facturas a los clientes a través de correo electrónico o WhatsApp, incluyendo los datos de contacto de soporte de la empresa.
- **FR11.** El sistema permitirá filtrar las facturas registradas por estado (pagada, impagada, recurrente) y por fecha.

### Seguimiento de Cobros y Notificaciones

- **FR12.** El sistema verificará de forma automática el estado de los pagos conectándose a fuentes de datos bancarias o pasarelas de pago (como Stripe).
- **FR13.** El sistema enviará recordatorios automáticos de impago a los clientes en periodos preestablecidos (ej. día 3, día 5 y de forma periódica si persiste el impago).
- **FR14.** El sistema enviará una notificación interna al empleado asignado al cliente cuando se realice un cobro o cuando un cliente entre en el flujo de impagos.

### Reportes e Integración

- **FR15.** El sistema proveerá un dashboard contable (visible solo para administradores) con reportes de facturación semanales, mensuales y anuales.
- **FR16.** El sistema dispondrá de una API documentada para permitir consultas futuras desde otros sistemas internos (como el producto principal Brain).
- **FR17.** El sistema enviará periódicamente copias de seguridad de los movimientos de facturación y cambios de permisos al superadministrador.

## Requisitos no funcionales

- **NFR1. Usabilidad y rapidez de operación**
  - **Descripción:** El sistema debe agilizar los procesos administrativos diarios minimizando el número de interacciones manuales necesarias.
  - **Métrica / Criterio de aceptación:** Las operaciones de creación de un nuevo cliente, generación de una factura y chequeo de estado deberán completarse en un tiempo inferior a 1 minuto y con el mínimo número de clics posible.
  - **Método de verificación:** Pruebas de usabilidad mediante cronometraje de las tareas críticas ejecutadas por usuarios del equipo de administración.
- **NFR2. Usabilidad responsive (Multidispositivo).**
  - **Descripción:** La aplicación debe poder ser operada desde cualquier lugar, ya sea desde un ordenador de sobremesa o dispositivos móviles.
  - **Métrica / Criterio de aceptación:** La interfaz debe ser una aplicación web completamente responsiva que se adapte sin pérdida de funcionalidad a resoluciones de teléfono móvil, tablet y escritorio.
  - **Método de verificación:** Pruebas manuales en diferentes resoluciones usando Chrome DevTools y pruebas funcionales en dispositivos físicos reales.
- **NFR3. Escalabilidad del sistema.**
  - **Descripción:** La arquitectura del sistema debe ser capaz de absorber el crecimiento proyectado de la empresa.
  - **Métrica / Criterio de aceptación:** El sistema de base de datos y la gestión documental deberán soportar la operativa de miles de clientes concurrentes sin degradación del rendimiento.
  - **Método de verificación:** Pruebas de carga y estrés en la base de datos simulando un volumen de 5.000 clientes activos.
- **NFR4. Persistencia legal y retención de datos.**
  - **Descripción:** Los datos de los clientes y las facturas no se deben eliminar de forma definitiva de la base de datos mientras exista obligación legal de conservarlos.
  - **Métrica / Criterio de aceptación:** Toda acción de borrado sobre un cliente o factura se ejecutará como un "borrado lógico" (modificación de estado) y los datos persistirán el tiempo exigido por la ley.
  - **Método de verificación:** Inspección del diseño de la base de datos y comprobación mediante pruebas de integración de que la consulta a la base de datos tras un borrado sigue retornando el registro oculto.
- **NFR5. Seguridad de acceso por roles.**
  - **Descripción:** La información de facturación es confidencial y debe estar compartimentada según la responsabilidad del usuario.
  - **Dependencias:** FR3, FR4.
  - **Métrica / Criterio de aceptación:** Un usuario con rol "Lector" tendrá el acceso denegado si intenta acceder mediante la interfaz o la API a los datos de un cliente que no tiene explícitamente asignado.
  - **Método de verificación:** Ejecución de pruebas unitarias sobre los endpoints de la API y pruebas manuales con perfiles limitados.
