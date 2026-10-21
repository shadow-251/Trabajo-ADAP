# Documento de Especificación de Requisitos de Software

**Proyecto:** Sistema Interno de Facturación y Seguimiento de Cobros (Turbine)

**Asignatura:** Análisis y Diseño de Aplicaciones / Ingeniería de Requisitos

### 1. Requisitos de Negocio (BR)

- BR1. Reducir la carga de trabajo manual en las tareas recurrentes de facturación y seguimiento de cobros, garantizando que las operaciones habituales se ejecuten en menos de un minuto.
- BR2. Disminuir los niveles de impago y los retrasos en las facturas mediante flujos automáticos de emisión, envío, recordatorios y recobro, manteniendo siempre el control y la supervisión por parte de la empresa.
- BR3. Diseñar un sistema capaz de gestionar de forma fluida miles de clientes activos y cientos de miles de facturas históricas sin requerir un aumento proporcional en el personal administrativo.
- BR4. Mantener la aplicación administrativa completamente independiente del producto principal de la empresa (*Brain Operative System*), asegurando su continuidad operativa ante cualquier evolución o cambio en la interfaz del producto comercial.

### 2. Requisitos de Usuario (UR)

- UR1. El gestor administrativo podrá registrar y actualizar la información comercial y fiscal de las empresas cliente, incluyendo múltiples personas de contacto y sus respectivos roles.
- UR2. El gestor administrativo podrá definir suscripciones recurrentes, aplicar cuotas basadas en rangos de facturación anual y tramitar solicitudes de descuento o excepciones comerciales.
- UR3. El usuario autorizado podrá emitir facturas recurrentes y puntuales, previsualizar los documentos en PDF y gestionar correcciones o facturas rectificativas con validez legal.
- UR4. El gestor podrá visualizar el estado de los cobros en tiempo real, si reciben transferencias bancarias asignar ese pago a una factura pendiente y gestionar disputas o incidencias con el cliente.
- UR5. El superadministrador podrá gestionar los usuarios internos, asignar carteras de clientes a responsables y configurar delegaciones temporales por ausencias o vacaciones.
- UR6. El administrador dispondrá de un cuadro de mando financiero con indicadores clave y la capacidad de exportar informes contables y paquetes periódicos para la gestoría.
- UR7. El usuario podrá iniciar sesión utilizando exclusivamente su email del dominio corporativo y dispondrá de un mecanismo para recuperar su contraseña en caso de olvido.

### 3. Requisitos Funcionales (FR)

- FR1. El sistema permitirá dar de alta, consultar, editar y borrar de forma lógica empresas cliente, registrando por separado sus datos comerciales (nombre comercial, responsable asignado) y fiscalidad (razón social, NIF/identificador fiscal, domicilio fiscal, país y moneda).
- FR2. El sistema permitirá asociar varias personas de contacto a una misma empresa cliente, especificando su rol (comercial, administración/facturación) y permitiendo inactivar contactos sin alterar el histórico de facturas enviadas a sus correos.
  - Dependencias: FR1.
- FR3. El sistema gestionará los distintos estados de un cliente (prospecto, pendiente de activación, activo, pausado, baja solicitada, inactivo, bloqueado) y ejecutará las restricciones correspondientes a cada estado.
  - Dependencias: FR1.
- FR4. El sistema aplicará borrado lógico al tramitar la baja de un cliente, garantizando la conservación de los datos fiscales y el historial de facturación durante los plazos legalmente exigidos.
  - Dependencias: FR1.
- FR5. El sistema comprobará la existencia previa del NIF o identificador fiscal antes de registrar un nuevo cliente y mostrará una advertencia si detecta coincidencia.
  - Dependencias: FR1.
- FR6. El sistema permitirá importar clientes desde archivos CSV, incluyendo previsualización, mapeo interactivo de columnas, validación de datos por fila y registro del resultado de la operación.
  - Dependencias: FR1, FR2, FR5.
- FR7. El sistema permitirá al superadministrador definir y actualizar los rangos de facturación anual y sus cuotas mensuales asociadas para la modalidad de suscripción.
- FR8. El sistema identificará los clientes cuya facturación anual deba revisarse, propondrá la nueva tarifa basada en los datos verificados y requerirá aprobación humana antes de aplicar la modificación a futuras facturas.
  - Dependencias: FR7, FR1.
- FR9. El sistema permitirá configurar acuerdos comerciales con fecha de inicio (actual o futura) y calculará de forma explícita el prorrateo correspondiente para altas producidas a mitad de mes.
  - Dependencias: FR1.
- FR10. El sistema permitirá pausar temporalmente una suscripción especificando motivo, fecha de inicio y fecha prevista de reactivación, deteniendo la emisión de cuotas sin alterar las deudas pasadas.
  - Dependencias: FR9.
- FR11. El sistema marcará como "Pendiente de aprobación" las facturas o cuotas que incluyan descuentos superiores a los umbrales configurados, precios modificados manualmente o condiciones de pago especiales.
  - Dependencias: FR9.
- FR12. El sistema ejecutará un proceso automático mensual (el día 1 de cada mes) para generar las facturas de las suscripciones activas.
  - Dependencias: FR9.
- FR13. El sistema permitirá redactar facturas puntuales compuestas por varias líneas (concepto, cantidad, precio unitario, descuento e impuesto).
  - Dependencias: FR1.
- FR14. El sistema ofrecerá un catálogo ligero de productos y servicios con precios e impuestos orientativos para agilizar la creación de facturas puntuales.
  - Dependencias: FR13.
- FR15. El sistema asignará un número correlativo e ininterrumpido por serie (ordinaria, rectificativa) en el momento exacto en que la factura pasa a estado emitida.
  - Dependencias: FR12, FR13.
- FR16. El sistema generará el archivo PDF de cada factura aplicando la plantilla estandarizada con la identidad visual de Turbine e impidiendo la emisión si faltan datos fiscales obligatorios.
  - Dependencias: FR15.
- FR17. El sistema bloqueará la alteración directa de facturas emitidas pertenecientes a periodos cerrados, exigiendo la emisión de una factura rectificativa o nota de crédito vinculada.
  - Dependencias: FR15.
- FR18. El sistema permitirá registrar facturas históricas marcándolas como "migradas", garantizando que no consuman numeración actual ni disparen envíos automáticos.
  - Dependencias: FR15.
- FR19. El sistema enviará las facturas emitidas por correo electrónico (al contacto de facturación) y/o WhatsApp (simulado), guardando traza de la fecha, canal, destinatario y resultado.
  - Dependencias: FR16, FR2.
- FR20. El sistema registrará los fallos de envío y rebotes de correo, notificando al responsable del cliente para corregir los datos de contacto y reenviar la factura sin duplicarla.
  - Dependencias: FR19.
- FR21. El sistema enviará recordatorios preventivos previos al vencimiento y ejecutará un flujo de recobro amigable con la cadencia que determine la configuración.
  - Dependencias: FR19.
- FR22. El sistema permitirá cambiar el estado de una factura a "En disputa" o "Seguimiento pausado" indicando motivo y fecha de revisión, pausando las comunicaciones automáticas de cobro.
  - Dependencias: FR21.
- FR23. El sistema procesará eventos de cobro desde Stripe, Revolut o extractos bancarios de forma idempotente, evitando duplicar registros de pago.
- FR24. El sistema vinculará las entradas de dinero con las facturas pendientes. Si existen diferencias de importe (pagos parciales o excesos), marcará el saldo pendiente y derivará el registro a conciliación manual.
  - Dependencias: FR23, FR15.
- FR25. El sistema permitirá tramitar la baja del cliente notificada desde el producto principal (*Brain*), cancelando la emisión de cuotas futuras pero manteniendo el flujo de cobro activo sobre facturas pendientes.
  - Dependencias: FR3, FR21.
- FR26. El sistema autenticará a los usuarios mediante credenciales seguras, restringiendo el registro e inicio de sesión a direcciones con el dominio oficial (`@turbine...`).
- FR27. El sistema aplicará tres niveles de permisos (Superadministrador, Administrador, Gestor operativo), limitando el acceso de los gestores únicamente a los clientes que tengan asignados.
  - Dependencias: FR26.
- FR28. El sistema permitirá traspasar temporalmente la cartera de clientes de un gestor a otro durante un periodo acotado por ausencias o vacaciones.
  - Dependencias: FR27.
- FR29. El sistema mostrará al usuario una vista inicial consolidada con sus tareas pendientes (vencimientos próximos, facturas impagadas, cobros a conciliar, rebotes y aprobaciones).
  - Dependencias: FR27.
- FR30. El sistema ofrecerá a los administradores métricas en tiempo real de facturación, cobros, volumen impagado, tiempo medio de cobro y proyección de ingresos recurrentes.
  - Dependencias: FR27, FR15, FR24.
- FR31. El sistema permitirá exportar listados a CSV/PDF y generará periódicamente un archivo estructurado (ZIP/CSV) con el libro de facturas emitidas, rectificativas y cobros para la gestoría fiscal.
  - Dependencias: FR30.
- FR32. El sistema guardará un registro detallado de cada acción relevante realizada (altas, modificaciones fiscales, emisiones, aprobaciones, conciliaciones y cambios de configuración) indicando usuario, timestamp y valores modificados.
  - Dependencias: FR26.
- FR33. El sistema expondrá endpoints documentados para permitir que aplicaciones autorizadas consulten el estado de deuda de un cliente y comuniquen solicitudes de baja.
  - Dependencias: FR3, FR24.
- FR34. El sistema permitirá a los usuarios solicitar el restablecimiento de su contraseña, generando y enviando al correo corporativo un enlace de recuperación temporal con caducidad configurada.
  - Dependencias: FR26

### 4. Requisitos No Funcionales (NFR)

- NFR1. Rendimiento en búsquedas y navegación.
  - Descripción: Las consultas, filtrados y búsquedas globales en el sistema deben responder de forma ágil para no penalizar la productividad del usuario.
  - Métrica / Criterio de aceptación: El 95% de las peticiones de búsqueda global y filtrado responderán en menos de 1000 ms sobre un volumen simulado de 2.500 clientes y 100.000 facturas en base de datos.
  - Método de verificación: Pruebas de carga y rendimiento mediante scripts automatizados con la herramienta k6 o Apache JMeter.
- NFR2. Agilidad operativa y usabilidad de la interfaz.
  - Descripción: La experiencia de usuario debe estar diseñada para minimizar los clics y el tiempo requerido en la ejecución de tareas administrativas habituales.
  - Métrica / Criterio de aceptación: Un usuario administrativo capacitado deberá poder completar el alta de un cliente estándar y la activación de su suscripción en un tiempo inferior a 60 segundos y realizando un máximo de 6 clics.
  - Método de verificación: Medición de tiempos mediante pruebas con usuarios reales (Task Analysis) y ejecución de suites de prueba end-to-end con Cypress o Playwright.
- NFR3. Usabilidad Responsive y Accesibilidad.
  - Descripción: La interfaz web del sistema debe ser plenamente utilizable en navegadores de ordenadores de escritorio, tabletas y smartphones.
  - Métrica / Criterio de aceptación: Cumplimiento del nivel AA de las pautas WCAG 2.1 en todas las vistas principales y ausencia de desbordamientos horizontales en rangos de resolución de 360px a 1920px.
  - Método de verificación: Evaluación automatizada con Axe-core / Lighthouse y verificación manual de la interfaz mediante Chrome DevTools en diferentes resoluciones.
- NFR4. Seguridad de acceso, autenticación y almacenamiento.
  - Descripción: El sistema debe proteger la información confidencial y evitar accesos no autorizados a datos fiscales y documentos.
  - Métrica / Criterio de aceptación: Las contraseñas se almacenarán utilizando algoritmos de hashing seguros (Bcrypt con factor de coste >= 12), las URLs de descarga de PDFs caducarán en un máximo de 15 minutos y se denegará el acceso a cualquier recurso sin una sesión autenticada con los permisos necesarios.
  - Método de verificación: Análisis estático de seguridad (SAST), escaneo de vulnerabilidades con OWASP ZAP y ejecución de pruebas unitarias sobre los interceptores de acceso.
- NFR5. Escalabilidad y arquitectura modular.
  - Descripción: La arquitectura de la aplicación debe ser capaz de absorber el crecimiento del volumen de usuarios e historial contable sin degradación de la infraestructura.
  - Métrica / Criterio de aceptación: El sistema soportará al menos 100 usuarios internos concurrentes sin sobrepasar el 80% de consumo de CPU/Memoria en servidor, e implementará paginación obligatoria en servidor con un límite máximo de 50 registros por petición.
  - Método de verificación: Pruebas de estrés y concurrencia simulada utilizando Apache JMeter sobre un entorno de preproducción.
- NFR6. Idempotencia y tolerancia a fallos en procesos en segundo plano.
  - Descripción: Las tareas automáticas periódicas (generación mensual de facturas, envíos masivos y conciliaciones) deben ser reintentables y seguras frente a caídas de red o servidor.
  - Métrica / Criterio de aceptación: La interrupción o reinicio del servidor durante la ejecución del lote mensual del día 1 permitirá una reejecución posterior con un 0% de duplicaciones en numeración de facturas o cobros.
  - Método de verificación: Pruebas de integración con inyección controlada de fallos simulando la caída del servicio a mitad de la ejecución en lote.
- NFR7. Protección de datos personales e integridad contable.
  - Descripción: El sistema implementará el principio de minimización de datos y permitirá la atención del derecho de supresión de datos personales sin romper la integridad contable.
  - Métrica / Criterio de aceptación: El proceso de anonimización de un contacto o cliente eliminará sus datos personales de las fichas activas en un 100%, conservando intactos los documentos PDF emitidos y las sumas contables.
  - Método de verificación: Ejecución de pruebas del procedimiento de supresión/anonimización y comprobación posterior de la consistencia de los datos contables en la base de datos.
- NFR8. Mantenibilidad, copias de seguridad y recuperación.
  - Descripción: El sistema debe contar con mecanismos para restaurar la información en caso de fallo crítico en la infraestructura.
  - Métrica / Criterio de aceptación: Copias de seguridad automáticas diarias con un tiempo máximo de recuperación de 2 horas y un punto de recuperación de pérdidas de un máximo de 24 horas.
  - Método de verificación: Simulación completa de restauración del sistema desde un archivo de backup en un entorno de prueba aislado.
- NFR9. Control de Concurrencia.
  - Descripción: El sistema debe detectar conflictos y evitar la sobrescritura silenciosa de datos cuando dos o más usuarios intenten modificar simultáneamente la misma entidad (ej. una factura o un cliente).
  - Métrica / Criterio de aceptación: El 100% de los conflictos de edición simultánea deberán ser bloqueados o notificados al segundo usuario antes de guardar.
  - Método de verificación: Pruebas de integración automatizadas simulando peticiones de guardado concurrentes al mismo recurso.
- NFR10. Precisión Decimal y Consistencia Financiera.
  - Descripción: El sistema debe asegurar la exactitud matemática en todos los importes, impuestos y totales, prohibiendo el uso de tipos de datos de coma flotante (floats) que generen descuadres por redondeos.
  - Métrica / Criterio de aceptación: 0 céntimos de discrepancia entre los totales calculados en la base de datos, el PDF generado y el cuadro de mando para el 100% de las facturas.
  - Método de verificación: Análisis estático de código para comprobar el uso de tipos de dato apropiados (ej. Decimal) y pruebas unitarias con sumas de precisión.
- NFR11. Seguridad y Ciclo de Vida de Sesiones.
  - Descripción: El sistema protegerá las sesiones activas, implementando expiración automática por inactividad y revocación de acceso inmediata si se desactiva un usuario.
  - Métrica / Criterio de aceptación: Cierre de sesión automático tras un periodo de inactividad configurable y denegación de acceso al 100% de las sesiones activas de usuarios desactivados administrativamente.
  - Método de verificación: Pruebas End-to-End (E2E) simulando tiempos de inactividad e invalidación manual del estado de usuario en la base de datos.