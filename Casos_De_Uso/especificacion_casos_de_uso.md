# Especificación de casos de uso

Los casos de uso que inicia una persona tienen una precondición común: el usuario ha iniciado sesión (CU-17) y solo actúa sobre los clientes que puede ver (un usuario operativo, los que tiene asignados o delegados, CU-22).

## Actores

| Actor | Descripción |
|---|---|
| Usuario operativo | Gestiona los clientes que tiene asignados o delegados. |
| Administrador | Hereda del usuario operativo. Ve todos los clientes, asigna y aprueba. |
| Superadministrador | Hereda del administrador. Mantiene la configuración global y consulta la auditoría. |
| Temporizador | Lanza los procesos programados: facturación mensual, recordatorios diarios y campaña de revisión anual. |
| Pasarela de cobro | Stripe simulado. Comunica los cobros. |
| Sistema externo | Brain u otro sistema que usa la API: consulta deudas y comunica bajas. |
| Servicio de mensajería | Actor secundario: correo electrónico y WhatsApp simulado. |

El cliente y la gestoría no son actores: reciben facturas y exportaciones, pero no usan el sistema.

## Diagramas

![Diagrama 1/4 - Clientes y relación comercial](Diagramas-PNG/diagrama_1_clientes_relacion_comercial.png)

![Diagrama 2/4 - Facturación](Diagramas-PNG/diagrama_2_facturacion.png)

![Diagrama 3/4 - Cobros](Diagramas-PNG/diagrama_3_cobros.png)

![Diagrama 4/4 - Acceso, seguimiento y administración](Diagramas-PNG/diagrama_4_seguimiento_administracion.png)

---

## Diagrama 1 - Clientes y relación comercial

### CU-01: Gestionar cliente

**Caso de uso:** Gestionar cliente.\
**ID:** CU-01\
**Actor principal:** Usuario operativo.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El usuario da de alta una empresa cliente o mantiene su ficha, con los datos comerciales y fiscales por separado y varios contactos, cada uno con su rol. La importación desde CSV es un caso aparte (CU-02).

**Interesados:**
- **Usuario operativo:** quiere dar de alta un cliente en poco tiempo.
- **Dirección de Turbine:** quiere evitar duplicados y fichas que bloqueen la facturación.
- **Cliente:** quiere facturas con sus datos correctos y enviadas a la persona adecuada.

**Precondiciones:**
- Para modificar un cliente, el usuario lo tiene asignado.

**Disparador:** Turbine cierra un acuerdo con una empresa o cambia algún dato de un cliente.

**Garantía de éxito:**
- La ficha queda guardada con al menos un contacto de facturación y un responsable, y el alta queda auditada.

**Garantía mínima:**
- No se crea un duplicado sin aviso y ningún cambio borra el histórico de facturas ni de envíos.

**Escenario principal de éxito:**
1. El usuario introduce los datos comerciales y fiscales del cliente (razón social, identificador fiscal, país y moneda).
2. El sistema valida el identificador fiscal según el país y busca duplicados.
3. El usuario añade los contactos con su rol (contratación, facturación o impagos) y su idioma.
4. El sistema comprueba que hay al menos un contacto de facturación.
5. El usuario confirma el alta.
6. El sistema guarda la ficha, asigna al usuario como responsable y registra el alta en la auditoría.

**Extensiones:**
- **1a.** El usuario modifica un cliente o inactiva un contacto: el sistema valida como en los pasos 2 y 4, conserva el histórico y audita el valor anterior y el nuevo.
- **2a.** El identificador fiscal ya existe: el sistema avisa del posible duplicado. Si es un cliente dado de baja que vuelve, el usuario reactiva su ficha con su histórico en lugar de crear otra; el sistema nunca fusiona fichas por su cuenta.
- **4a.** No hay contacto de facturación: el sistema guarda la ficha, la marca como incompleta para facturar e indica qué falta.

### CU-02: Importar clientes desde CSV

**Caso de uso:** Importar clientes desde CSV.\
**ID:** CU-02\
**Actor principal:** Usuario operativo.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El usuario da de alta o actualiza muchos clientes a la vez a partir de un fichero CSV. Antes de confirmar ve una previsualización, asocia cada columna con su campo y revisa los errores fila a fila, para que un fichero mal formado no meta cientos de clientes corruptos. La importación queda registrada como una operación con su resumen.

**Interesados:**
- **Usuario operativo:** quiere cargar muchos clientes de golpe sin darlos de alta uno a uno.
- **Dirección de Turbine:** quiere que un fichero erróneo no meta clientes corruptos ni duplicados.

**Precondiciones:**
- Ninguna además de la común.

**Disparador:** El usuario tiene un fichero CSV con clientes que dar de alta o actualizar.

**Garantía de éxito:**
- Las filas aceptadas quedan como clientes creados o actualizados, y la importación queda registrada con su resumen de creados, actualizados, rechazados y motivos.

**Garantía mínima:**
- Ninguna fila entra sin pasar las mismas validaciones que un alta manual (CU-01), y una importación cancelada no deja clientes a medias.

**Escenario principal de éxito:**
1. El usuario sube el fichero CSV.
2. El sistema muestra una previsualización de las primeras filas.
3. El usuario asocia cada columna del fichero con un campo de la ficha de cliente.
4. El sistema valida cada fila como en un alta manual y busca duplicados por identificador fiscal.
5. El sistema muestra cuántas filas son válidas, cuántas tienen errores y por qué.
6. El usuario confirma la importación.
7. El sistema crea los clientes, asigna al usuario como responsable y registra la importación con su resumen.

**Extensiones:**
- **2a.** El fichero no se puede leer (formato o codificación incorrectos): el sistema lo rechaza e indica el motivo, sin importar nada.
- **4a.** Una fila coincide por identificador fiscal con un cliente existente: el sistema avisa del posible duplicado y el usuario decide si actualiza esa ficha, si la tiene asignada, o descarta la fila. El sistema nunca fusiona clientes por su cuenta.
- **5a.** Hay filas con errores: el usuario decide si corrige el fichero y vuelve a subirlo (paso 1) o importa solo las filas válidas.
- **6a.** El fichero es muy grande: el sistema hace la importación en segundo plano y avisa cuando termina.
- **\*a.** El usuario cancela antes de confirmar: el sistema no importa nada.

### CU-03: Gestionar suscripción

**Caso de uso:** Gestionar suscripción.\
**ID:** CU-03\
**Actor principal:** Usuario operativo.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El usuario configura la relación comercial que origina la facturación recurrente (plan, conceptos, tarifa, condiciones de pago y fecha de inicio) y la modifica cuando cambian las condiciones. Es mensual y sin permanencia. La revisión anual de rango y las pausas tienen su propio caso de uso (CU-05 y CU-04).

**Interesados:**
- **Usuario operativo:** quiere ver lo que se facturará antes de confirmar.
- **Dirección de Turbine:** quiere que no se aplique ningún precio ni descuento sin autorizar.
- **Cliente:** quiere que se le facture lo pactado.

**Precondiciones:**
- El usuario tiene asignado el cliente y las tarifas por rango están configuradas.

**Disparador:** Se cierra un acuerdo con el cliente o cambian sus condiciones.

**Garantía de éxito:**
- La suscripción queda activa, o pendiente de activación si empieza más adelante, con su tarifa, sus condiciones y su fecha de efecto.

**Garantía mínima:**
- Ningún cambio sujeto a aprobación se aplica sin ella, ninguno altera facturas emitidas y todos quedan auditados.

**Escenario principal de éxito:**
1. El usuario indica el plan, los conceptos, la fecha de inicio y las condiciones de pago.
2. El sistema comprueba que el cliente tiene los datos fiscales y el contacto de facturación.
3. El sistema propone la tarifa del rango de facturación anual del cliente y el vencimiento por defecto.
4. El sistema muestra el prorrateo hasta fin de mes y la primera mensualidad, que se facturarán el día 1 siguiente.
5. El usuario confirma.
6. El sistema activa la suscripción y registra el alta en la auditoría.

**Extensiones:**
- **1a.** El usuario modifica una suscripción: indica el cambio y su fecha de efecto; el sistema muestra cómo afecta a la próxima factura y continúa en el paso 5.
- **2a.** Faltan datos para facturar: el sistema bloquea la activación e indica qué falta.
- **5a.** El cambio está sujeto a aprobación (descuento, primer mes gratis, precio pactado o facturación sin prorrateo): se ejecuta CU-11 Solicitar aprobación y el cambio se aplica al aprobarse (CU-12).
- **6a.** La fecha de inicio es futura: el sistema deja la suscripción pendiente de activación hasta esa fecha.

### CU-04: Pausar / reanudar suscripción

**Caso de uso:** Pausar / reanudar suscripción.\
**ID:** CU-04\
**Actor principal:** Usuario operativo.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El usuario pausa temporalmente la suscripción de un cliente por un acuerdo con él, con motivo, fecha de inicio y, si se conoce, fecha prevista de reactivación, y la reanuda cuando termina. Durante la pausa no se generan mensualidades, pero las facturas pendientes siguen en recobro. Pausar no es dar de baja: el cliente sigue existiendo y la relación comercial continúa.

**Interesados:**
- **Usuario operativo:** quiere dejar de facturar temporalmente sin perder la relación con el cliente.
- **Dirección de Turbine:** quiere que la pausa no borre la deuda y que nadie olvide reanudarla.
- **Cliente:** quiere que no se le facture mientras dura la pausa acordada.

**Precondiciones:**
- El cliente tiene una suscripción activa y el usuario lo tiene asignado.

**Disparador:** Turbine acuerda con el cliente una pausa temporal del servicio.

**Garantía de éxito:**
- La suscripción queda pausada con su motivo y sus fechas y, al reanudarse, vuelve a generar mensualidades desde la fecha de reanudación.

**Garantía mínima:**
- La pausa nunca cancela las facturas pendientes ni sus recordatorios, y queda auditada.

**Escenario principal de éxito:**
1. El usuario indica que quiere pausar la suscripción de un cliente.
2. El usuario introduce el motivo, la fecha de inicio y la fecha prevista de reactivación.
3. El sistema muestra qué mensualidades dejarán de generarse y qué facturas pendientes seguirán en recobro.
4. El usuario confirma la pausa.
5. El sistema pausa la suscripción desde la fecha de inicio y registra la pausa en la auditoría.
6. Al llegar la fecha prevista de reactivación, el sistema avisa al responsable.
7. El usuario confirma la reanudación.
8. El sistema reanuda la suscripción y vuelve a generar mensualidades desde esa fecha.

**Extensiones:**
- **1a.** La suscripción ya está pausada y el usuario quiere reanudarla antes de tiempo: indica la fecha de reanudación y continúa en el paso 8.
- **2a.** No se conoce la fecha de reactivación: la pausa queda abierta y el sistema no avisa; el usuario la reanuda cuando corresponda (extensión 1a).
- **6a.** La política configurada no exige confirmación: el sistema reanuda la suscripción en esa fecha y avisa al responsable.
- **7a.** El cliente pide alargar la pausa: el usuario indica la nueva fecha prevista y el sistema vuelve a avisar cuando llegue.

### CU-05: Revisar rango de tarifa anual

**Caso de uso:** Revisar rango de tarifa anual.\
**ID:** CU-05\
**Actor principal:** Temporizador, que abre la campaña de revisión.\
**Actor secundario:** Usuario operativo, responsable de cada cliente, que aporta el dato y confirma la propuesta.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** Una vez al año, o en la fecha configurada, el sistema abre una campaña para revisar si cada cliente ha cambiado de rango de facturación anual, que es lo que determina su tarifa. Para cada cliente, el responsable aporta la facturación anual verificada y el sistema propone el rango que corresponde. Si el rango cambia, el cambio necesita aprobación y el nuevo precio se aplica desde su fecha de efecto, sin reescribir meses anteriores.

**Interesados:**
- **Dirección de Turbine:** quiere actualizar las tarifas cada año sin que un dato mal interpretado cambie cien clientes.
- **Usuario operativo:** quiere saber qué clientes le toca revisar y tener la propuesta ya calculada.
- **Cliente:** quiere que un cambio de precio no afecte a las facturas ya emitidas.

**Precondiciones:**
- La tabla de rangos y tarifas está configurada (CU-23).

**Disparador:** Llega la fecha de la campaña de revisión anual.

**Garantía de éxito:**
- Cada cliente revisado tiene su rango confirmado o un cambio de rango pendiente de aprobación, con el dato, la fuente, la fecha y quién lo revisó.

**Garantía mínima:**
- Ninguna tarifa cambia sin aprobación y ningún cambio altera facturas ya emitidas.

**Escenario principal de éxito:**
1. El Temporizador inicia la campaña de revisión anual.
2. El sistema lista los clientes con suscripción activa cuya facturación anual toca revisar y crea una revisión pendiente para cada uno en el trabajo pendiente de su responsable (CU-18).
3. El usuario abre la revisión de un cliente.
4. El usuario introduce la facturación anual verificada, con su fuente y su fecha.
5. El sistema propone el rango y la tarifa que corresponden según la tabla vigente.
6. El usuario confirma la propuesta.
7. El sistema cierra la revisión y registra el dato, la fuente, quién la hizo y cuándo.

**Extensiones:**
- **3a.** El usuario corrige una revisión ya cerrada porque el dato era erróneo: el sistema guarda la corrección con trazabilidad y calcula el impacto potencial, pero no emite facturas retroactivas; cualquier regularización es una acción explícita y aprobada.
- **4a.** El dato llega de la fuente configurada (simulada): el sistema lo rellena y el usuario solo lo verifica.
- **4b.** El dato no está disponible: la revisión queda pendiente y sigue en el trabajo pendiente del usuario.
- **6a.** El rango propuesto es distinto del actual: se ejecuta CU-11 Solicitar aprobación. Al aprobarse (CU-12), el sistema aplica la nueva tarifa desde su fecha de efecto y registra quién la aprobó, cuándo, y el precio anterior y el nuevo.

### CU-06: Tramitar baja

**Caso de uso:** Tramitar baja.\
**ID:** CU-06\
**Actor principal:** Usuario operativo (o Sistema externo, cuando la solicitud llega por la API de Brain).\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** Se registra la baja de un cliente y el sistema deja de facturarle desde la fecha efectiva. La baja corta las mensualidades nuevas, pero no la deuda: lo pendiente sigue en recobro.

**Interesados:**
- **Dirección de Turbine:** quiere dejar de facturar sin renunciar a cobrar lo adeudado.
- **Cliente:** quiere que no se le facture después de la baja.
- **Brain:** quiere conocer la deuda del cliente.
- **Gestoría:** quiere que se conserven los datos contables.

**Precondiciones:**
- El cliente tiene una suscripción activa o pausada; por API, el sistema externo está autenticado.

**Disparador:** El cliente solicita la baja del servicio.

**Garantía de éxito:**
- La suscripción se cierra en la fecha efectiva y las facturas pendientes siguen en recobro.

**Garantía mínima:**
- No se borra ningún dato contable y ninguna petición externa cierra la recurrencia saltándose las reglas.

**Escenario principal de éxito:**
1. El usuario registra la solicitud de baja con su fecha efectiva.
2. El sistema muestra las facturas emitidas y no cobradas del cliente.
3. El usuario confirma la baja.
4. El sistema programa el cierre de la suscripción: el cliente figura como «baja solicitada» hasta la fecha efectiva y como inactivo después.
5. El sistema registra la baja en la auditoría y notifica al responsable.

**Extensiones:**
- **1a.** La solicitud llega por API: el sistema la registra como pendiente, responde con las obligaciones pendientes y crea una tarea para el responsable, que continúa en el paso 2.
- **1b.** El sistema externo solo consulta la deuda: el sistema devuelve si hay obligaciones pendientes, con el detalle mínimo autorizado.
- **1c.** La petición por API no está autenticada: el sistema la rechaza y registra el intento.
- **2a.** Hay facturas pendientes: el sistema avisa de que su recobro continuará, sin bloquear la baja.

---

## Diagrama 2 - Facturación

### CU-07: Generar facturas recurrentes

**Caso de uso:** Generar facturas recurrentes.\
**ID:** CU-07\
**Actor principal:** Temporizador.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El día 1 de cada mes, el sistema genera y emite (incluye CU-09) la mensualidad de cada suscripción activa con la tarifa y los descuentos vigentes. Se puede relanzar sin duplicar, y cada excepción llega a su responsable sin detener el resto.

**Interesados:**
- **Dirección de Turbine:** quiere facturar a tiempo sin intervención y saber qué pasó en cada ejecución.
- **Usuario operativo:** quiere recibir solo las excepciones.
- **Cliente:** quiere una única factura correcta al mes.

**Precondiciones:**
- Las tarifas y la serie de facturación están configuradas.

**Disparador:** Llega el día 1 del mes o el sistema se recupera de una ejecución incompleta.

**Garantía de éxito:**
- Cada suscripción activa tiene exactamente una mensualidad emitida y la ejecución queda registrada con su resumen.

**Garantía mínima:**
- Ninguna mensualidad se duplica aunque se relance el proceso, y el fallo de un cliente no impide procesar al resto.

**Escenario principal de éxito:**
1. El Temporizador inicia la generación del mes.
2. El sistema selecciona las suscripciones activas, sin las pausadas, las pendientes de activación ni las dadas de baja.
3. Para cada una, el sistema comprueba que no existe ya la mensualidad del periodo.
4. El sistema compone la factura con la tarifa y los descuentos vigentes, el prorrateo pendiente si lo hay y el vencimiento pactado.
5. El sistema emite la factura (incluye CU-09 Emitir factura).
6. El sistema repite los pasos 3 a 5 con cada suscripción.
7. El sistema registra el resumen de la ejecución y notifica las excepciones a cada responsable.

**Extensiones:**
- **1a.** El sistema estaba caído a la hora programada: al recuperarse, procesa el periodo pendiente; el paso 3 evita duplicados.
- **3a.** La mensualidad ya existe: el sistema la omite.
- **5a.** La emisión falla (por ejemplo, falta el contacto de facturación): la factura queda en borrador, el sistema avisa al responsable y sigue con la siguiente suscripción.
- **5b.** La factura queda pendiente de aprobación: el sistema lo anota en el resumen y continúa.

### CU-08: Crear factura puntual

**Caso de uso:** Crear factura puntual.\
**ID:** CU-08\
**Actor principal:** Usuario operativo.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El usuario prepara una factura no recurrente (instalación, consultoría, formación…) con una o varias líneas, apoyándose en un catálogo de conceptos, y la emite (incluye CU-09). El borrador se puede corregir y no consume número.

**Interesados:**
- **Usuario operativo:** quiere facturar sin reescribir conceptos.
- **Dirección de Turbine:** quiere evitar importes absurdos por error.
- **Cliente:** quiere una factura clara y correcta.

**Precondiciones:**
- El usuario tiene asignado el cliente.

**Disparador:** Turbine presta un servicio puntual que hay que facturar.

**Garantía de éxito:**
- La factura queda emitida y enviada, con sus bases, impuestos y total exactos.

**Garantía mínima:**
- El borrador se conserva aunque se interrumpa la edición.

**Escenario principal de éxito:**
1. El usuario crea una factura puntual para un cliente.
2. El sistema crea un borrador sin número con los datos y las condiciones del cliente.
3. El usuario añade líneas desde el catálogo, con su cantidad y, si aplica, un descuento.
4. El sistema recalcula bases, impuestos y total, y guarda el borrador.
5. El sistema muestra la vista previa de la factura.
6. El usuario confirma la emisión.
7. El sistema emite la factura (incluye CU-09 Emitir factura).

**Extensiones:**
- **3a.** El concepto no está en el catálogo: el usuario escribe la descripción, el precio y el impuesto.
- **5a.** El importe se sale del patrón del cliente: el sistema avisa sin bloquear; el usuario confirma o vuelve al paso 3.
- **6a.** El usuario deja la emisión para más tarde: el borrador se conserva sin número.
- **\*a.** Otro usuario modificó el mismo borrador a la vez: el sistema detecta el conflicto y no sobrescribe sus cambios.

### CU-09: Emitir factura

**Caso de uso:** Emitir factura.\
**ID:** CU-09\
**Actor principal:** El del caso de uso que lo incluye (Temporizador en CU-07, Usuario operativo en CU-08 o Administrador en CU-10).\
**Actor secundario:** Servicio de mensajería.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Subfunción.

**Descripción:** Convierte un borrador en factura emitida: valida los datos fiscales, asigna el número, genera el PDF y envía la factura. Lo incluyen CU-07, CU-08 y CU-10. Una factura emitida ya no se edita: se corrige con una rectificativa (CU-10).

**Interesados:**
- **Gestoría:** quiere una numeración única por serie y datos fiscales completos.
- **Cliente:** quiere recibir la factura correcta.
- **Usuario operativo:** quiere saber si la factura se envió.

**Precondiciones:**
- Existe un borrador con al menos una línea y su serie está configurada.

**Disparador:** El caso de uso que lo incluye pide emitir la factura.

**Garantía de éxito:**
- La factura queda emitida con número definitivo y PDF conservado, y el envío queda registrado con su resultado.

**Garantía mínima:**
- Nunca se repite un número ni se emite dos veces la misma factura; si el envío falla, la factura sigue emitida y consta como no enviada.

**Escenario principal de éxito:**
1. El sistema comprueba los datos fiscales imprescindibles del emisor y del cliente.
2. El sistema comprueba las reglas de aprobación.
3. El sistema asigna de forma atómica el siguiente número de la serie y la fecha de emisión.
4. El sistema congela los datos de la factura, genera el PDF y lo conserva.
5. El sistema envía la factura a los contactos de facturación, en su idioma, a través del Servicio de mensajería.
6. El sistema registra el envío (canal, destinatarios, fecha y resultado) y la emisión en la auditoría.

**Extensiones:**
- **1a.** Falta un dato fiscal: el sistema bloquea la emisión, indica cuál y la factura sigue en borrador.
- **2a.** Se cumple una regla de aprobación (importe modificado a mano, descuento extraordinario, vencimiento atípico o importe alto): se ejecuta CU-11 Solicitar aprobación; al aprobarse (CU-12), la emisión sigue en el paso 3.
- **3a.** La emisión ya está en curso por una petición repetida: el sistema no inicia otra.
- **5a.** El envío falla o rebota: el sistema registra el fallo y se ejecuta CU-16 Gestionar incidencia de facturación, que avisa al responsable.
- **\*a.** Error inesperado en los pasos 3 o 4: el sistema deshace la emisión y la factura sigue en borrador sin consumir número.

### CU-10: Emitir factura rectificativa

**Caso de uso:** Emitir factura rectificativa.\
**ID:** CU-10\
**Actor principal:** Administrador.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El administrador corrige una factura emitida sin editarla: el sistema genera una rectificativa vinculada a la original, en su propia serie, y la emite (incluye CU-09).

**Interesados:**
- **Gestoría:** quiere que la corrección siga el procedimiento legal y quede vinculada a la factura original.
- **Cliente:** quiere que su saldo sea correcto.

**Precondiciones:**
- La factura original está emitida y hay una serie de rectificativas configurada.

**Disparador:** Se detecta un error en una factura emitida o hay que devolver parte del importe.

**Garantía de éxito:**
- La rectificativa queda emitida y vinculada, y el saldo del cliente refleja la corrección.

**Garantía mínima:**
- La factura original nunca se modifica ni se borra.

**Escenario principal de éxito:**
1. El administrador indica la factura que quiere rectificar.
2. El sistema propone una rectificativa a partir de sus líneas.
3. El administrador indica qué se rectifica y el motivo.
4. El sistema calcula las diferencias y muestra la vista previa.
5. El administrador confirma.
6. El sistema emite la rectificativa (incluye CU-09 Emitir factura), la vincula a la original y recalcula el saldo del cliente.

**Extensiones:**
- **1a.** La factura sigue en borrador: el sistema indica que se corrige directamente, sin rectificativa.
- **6a.** El importe supera el umbral de aprobación: la rectificativa queda pendiente de aprobación (CU-09, extensión 2a) y se emite cuando se aprueba.

### CU-11: Solicitar aprobación

**Caso de uso:** Solicitar aprobación.\
**ID:** CU-11\
**Actor principal:** El del caso de uso desde el que se añade (CU-03, CU-09 o CU-05).\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Subfunción.

**Descripción:** Retiene una operación económica sensible hasta que la apruebe quien corresponda. Se añade con «extend» desde CU-03, CU-09 y CU-05. La aprobación es asíncrona y se resuelve en CU-12.

**Interesados:**
- **Dirección de Turbine:** quiere que ningún descuento ni cambio de importe se aplique sin control.
- **Solicitante:** quiere saber por qué su operación está pendiente.

**Precondiciones:**
- El caso base ha detectado que la operación cumple una regla de aprobación.

**Disparador:** CU-03, CU-09 o CU-05 detectan una operación que requiere aprobación.

**Garantía de éxito:**
- Existe una solicitud pendiente, con su motivo y su aprobador, y la operación queda retenida.

**Garantía mínima:**
- La operación nunca se aplica sin aprobación.

**Escenario principal de éxito:**
1. El sistema identifica la regla que se cumple y el nivel de aprobación necesario.
2. El sistema crea la solicitud con la operación, el motivo y el solicitante, y retiene la operación.
3. El sistema asigna la solicitud a un aprobador del nivel requerido, distinto del solicitante si la regla lo exige.
4. El sistema notifica al aprobador e informa al solicitante.

**Extensiones:**
- **2a.** Ya hay una solicitud pendiente para esa operación: el sistema no crea otra.
- **3a.** El aprobador está ausente: el sistema asigna la solicitud a su suplente.

### CU-12: Resolver solicitud de aprobación

**Caso de uso:** Resolver solicitud de aprobación.\
**ID:** CU-12\
**Actor principal:** Administrador.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El aprobador revisa sus solicitudes pendientes, con el motivo de cada una, y las aprueba o rechaza con un comentario, también desde el móvil.

**Interesados:**
- **Solicitante:** quiere conocer la decisión.
- **Dirección de Turbine:** quiere separación de funciones y trazabilidad.

**Precondiciones:**
- El aprobador tiene solicitudes pendientes asignadas.

**Disparador:** El aprobador recibe la notificación de una solicitud.

**Garantía de éxito:**
- La solicitud queda resuelta con aprobador, fecha y comentario, y si se aprueba la operación continúa.

**Garantía mínima:**
- Nadie aprueba su propia solicitud cuando la regla lo impide, y toda decisión queda auditada.

**Escenario principal de éxito:**
1. El aprobador consulta sus solicitudes pendientes.
2. El sistema las muestra con la operación, el importe, el solicitante y el motivo.
3. El aprobador aprueba una solicitud y, si quiere, añade un comentario.
4. El sistema registra la decisión, reanuda la operación retenida y notifica al solicitante.

**Extensiones:**
- **1a.** Un administrador reasigna la solicitud a otro aprobador: el sistema la reasigna y lo registra.
- **3a.** El aprobador la rechaza con un motivo: la operación vuelve a su estado anterior y el sistema avisa al solicitante.
- **4a.** La operación ya no puede continuar (por ejemplo, falta un dato fiscal): el sistema avisa al solicitante para que la corrija.

---

## Diagrama 3 - Cobros

### CU-13: Registrar cobro

**Caso de uso:** Registrar cobro.\
**ID:** CU-13\
**Actor principal:** Pasarela de cobro.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El sistema recibe un movimiento de cobro, lo registra una sola vez e intenta asociarlo a su factura. Si la coincidencia es clara lo concilia; si no, lo deja para conciliación manual (CU-15).

**Interesados:**
- **Dirección de Turbine:** quiere no contar dos veces un cobro.
- **Usuario operativo:** quiere revisar solo los cobros dudosos.
- **Cliente:** quiere que no se le reclame lo que ya ha pagado.

**Precondiciones:**
- La fuente de cobro está configurada y autenticada.

**Disparador:** La pasarela notifica un pago.

**Garantía de éxito:**
- El movimiento queda registrado y, si la coincidencia es clara, la factura queda pagada y sus recordatorios se detienen.

**Garantía mínima:**
- Un evento duplicado nunca cuenta dos veces y ningún movimiento se pierde.

**Escenario principal de éxito:**
1. La pasarela comunica un movimiento con su identificador, importe, fecha, moneda y referencia.
2. El sistema comprueba que no lo había registrado antes y lo registra.
3. El sistema localiza la factura por la referencia y comprueba que el importe coincide con su saldo pendiente.
4. El sistema concilia el cobro, marca la factura como pagada y detiene sus recordatorios.
5. El sistema notifica el pago al responsable del cliente.

**Extensiones:**
- **2a.** El movimiento ya estaba registrado: el sistema no lo vuelve a contar y confirma a la pasarela que ya está procesado.
- **3a.** La referencia no identifica una única factura (falta o hay varias candidatas): el sistema no elige ninguna y deja el movimiento para conciliación manual (CU-15).
- **3b.** El importe no coincide (pago parcial o de más): el sistema propone la asociación sin marcar la factura como pagada y la deja para conciliación manual.
- **4a.** La política configurada exige validación humana: el sistema deja la conciliación propuesta para CU-15.

### CU-14: Enviar recordatorios de cobro

**Caso de uso:** Enviar recordatorios de cobro.\
**ID:** CU-14\
**Actor principal:** Temporizador.\
**Actor secundario:** Servicio de mensajería.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** Cada día, el sistema envía los recordatorios que tocan a las facturas pendientes según una cadencia configurable referida al vencimiento. Por defecto: aviso dos días antes, aviso el día del vencimiento y avisos de impago cada 2 o 3 días.

**Interesados:**
- **Dirección de Turbine:** quiere cobrar a tiempo sin perseguir cada factura.
- **Cliente:** quiere no recibir avisos indebidos ni repetidos.

**Precondiciones:**
- La cadencia, las plantillas y el calendario laboral están configurados.

**Disparador:** El Temporizador lanza la ejecución diaria.

**Garantía de éxito:**
- Cada factura pendiente recibe los recordatorios que le tocan, en el idioma del contacto, y la ejecución queda registrada.

**Garantía mínima:**
- No se avisa de facturas pagadas o con el seguimiento pausado ni antes del vencimiento pactado, y ningún aviso se envía dos veces.

**Escenario principal de éxito:**
1. El Temporizador inicia la ejecución diaria.
2. El sistema selecciona las facturas emitidas con saldo pendiente y sin el seguimiento pausado.
3. Para cada una, el sistema calcula qué recordatorio le toca hoy según su vencimiento.
4. El sistema envía el recordatorio con su plantilla, en el idioma del contacto, a través del Servicio de mensajería.
5. El sistema notifica al responsable las facturas que acaban de vencer y registra el resumen.

**Extensiones:**
- **1a.** El sistema estaba caído a la hora programada: al recuperarse, envía los recordatorios pendientes del día sin repetir los ya enviados.
- **3a.** El aviso cae en festivo o fin de semana: el sistema lo traslada al siguiente día laborable.
- **4a.** El envío falla: el sistema lo registra, avisa al responsable y sigue con la siguiente factura.

### CU-15: Conciliar cobro manualmente

**Caso de uso:** Conciliar cobro manualmente.\
**ID:** CU-15\
**Actor principal:** Usuario operativo.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El usuario resuelve los cobros que el sistema no pudo conciliar solo: los asocia a su factura, valida las conciliaciones propuestas o deshace una errónea. Si algo no cuadra, se abre una incidencia (CU-16).

**Interesados:**
- **Usuario operativo:** quiere resolver rápido los cobros dudosos.
- **Dirección de Turbine:** quiere que cada factura pagada tenga su cobro y que los errores se puedan deshacer dejando rastro.

**Precondiciones:**
- Hay movimientos pendientes de conciliar de clientes del usuario.

**Disparador:** El usuario ve cobros por conciliar en su trabajo pendiente (CU-18).

**Garantía de éxito:**
- El movimiento queda asociado a su factura, con quién lo concilió y cuándo, y la factura refleja su saldo real.

**Garantía mínima:**
- Nada se borra: una conciliación deshecha queda registrada, y dos usuarios no pueden conciliar a la vez el mismo movimiento.

**Escenario principal de éxito:**
1. El usuario consulta los movimientos pendientes de conciliar.
2. El sistema los muestra con sus datos y las facturas candidatas.
3. El usuario indica a qué factura corresponde un movimiento.
4. El sistema comprueba que el importe y la moneda cuadran con el saldo pendiente.
5. El usuario confirma.
6. El sistema concilia, marca la factura como pagada, detiene sus recordatorios y registra la conciliación.

**Extensiones:**
- **1a.** El usuario deshace una conciliación errónea, con su motivo: el sistema la revierte sin borrar el movimiento y reactiva los recordatorios si procede.
- **3a.** Hay una conciliación propuesta: el usuario la valida y continúa en el paso 6.
- **4a.** El importe no cuadra (pago parcial o de más) o el movimiento no corresponde a ninguna factura: el sistema no marca la factura como pagada, registra la diferencia y se ejecuta CU-16 Gestionar incidencia de facturación.
- **5a.** Otro usuario ya concilió el movimiento: el sistema no aplica la segunda conciliación y avisa.

### CU-16: Gestionar incidencia de facturación

**Caso de uso:** Gestionar incidencia de facturación.\
**ID:** CU-16\
**Actor principal:** Usuario operativo.\
**Actor secundario:** Servicio de mensajería.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El usuario abre y sigue una incidencia interna sobre un cliente o una factura (NIF incorrecto, importe reclamado, correo no recibido, pago no localizado), con responsable, estado, prioridad y comentarios. Puede pausar los recordatorios mientras se estudia. También se añade con «extend» desde CU-09 y CU-15.

**Interesados:**
- **Usuario operativo:** quiere seguir cada problema sin perderlo.
- **Cliente:** quiere no recibir recordatorios mientras reclama.

**Precondiciones:**
- El usuario tiene asignado el cliente.

**Disparador:** El usuario detecta un problema, o el sistema lo detecta en CU-09 o CU-15.

**Garantía de éxito:**
- La incidencia queda registrada con su responsable y, al cerrarse, con su resolución.

**Garantía mínima:**
- Ninguna incidencia se cierra sin resolución y toda pausa de recordatorios tiene fecha de revisión.

**Escenario principal de éxito:**
1. El usuario abre una incidencia sobre un cliente y, si procede, una factura, con su tipo, descripción y prioridad.
2. El sistema la asigna al responsable del cliente y le notifica.
3. El usuario indica si pausa los recordatorios de la factura, con su motivo y fecha de revisión.
4. El sistema pausa el seguimiento sin cambiar el estado de cobro de la factura.
5. El usuario añade comentarios y adjuntos y actualiza el estado a medida que avanza.
6. El usuario cierra la incidencia con su resolución.
7. El sistema registra el cierre y reanuda los recordatorios si estaban pausados.

**Extensiones:**
- **1a.** El sistema abre la incidencia desde CU-09 (fallo de envío) o CU-15 (discrepancia), asociada a la factura, y continúa en el paso 2.
- **3a.** La incidencia no afecta al cobro: continúa en el paso 5.
- **5a.** Es una incidencia de entrega: el usuario corrige el contacto y el sistema reenvía la misma factura, sin nuevo número, a través del Servicio de mensajería.
- **6a.** No se indica la resolución: el sistema no permite cerrar la incidencia.
- **\*a.** Llega la fecha de revisión de la pausa: el sistema avisa al responsable.

---

## Diagrama 4 - Acceso, seguimiento y administración

### CU-17: Iniciar sesión

**Caso de uso:** Iniciar sesión.\
**ID:** CU-17\
**Actor principal:** Usuario operativo (también el administrador y el superadministrador, que heredan de él).\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Subfunción.

**Descripción:** El usuario se identifica con su correo corporativo y su contraseña para acceder al sistema. La sesión caduca por inactividad y el usuario puede cerrarla cuando quiera. Es la precondición común del resto de casos de uso que inicia una persona.

**Interesados:**
- **Usuario operativo:** quiere entrar rápido, sin bloqueos innecesarios.
- **Dirección de Turbine:** quiere que solo accedan usuarios internos activos y que una sesión olvidada no quede abierta.

**Precondiciones:**
- El usuario tiene una cuenta activa, creada por invitación (CU-21).

**Disparador:** El usuario quiere usar el sistema.

**Garantía de éxito:**
- El usuario tiene una sesión abierta con los permisos de su rol y de sus clientes asignados, y el acceso queda registrado.

**Garantía mínima:**
- Nadie entra con credenciales incorrectas ni con una cuenta desactivada, y los intentos fallidos quedan registrados.

**Escenario principal de éxito:**
1. El usuario introduce su correo y su contraseña.
2. El sistema comprueba que el correo pertenece al dominio corporativo autorizado.
3. El sistema comprueba la contraseña, que guarda cifrada y nunca en texto plano.
4. El sistema comprueba que la cuenta está activa.
5. El sistema abre una sesión con los permisos del rol del usuario y registra el acceso.

**Extensiones:**
- **2a.** El correo no es del dominio autorizado: el sistema rechaza el acceso con un mensaje genérico.
- **3a.** La contraseña es incorrecta: el sistema rechaza el acceso sin indicar qué dato falla y registra el intento. Tras varios intentos fallidos seguidos, retrasa temporalmente los siguientes.
- **4a.** La cuenta está desactivada: el sistema rechaza el acceso.
- **\*a.** El usuario cierra la sesión: el sistema la invalida.
- **\*b.** Pasa el tiempo máximo de inactividad: el sistema cierra la sesión.
- **\*c.** Un administrador desactiva la cuenta (CU-21): el sistema invalida sus sesiones activas.

### CU-18: Consultar trabajo pendiente

**Caso de uso:** Consultar trabajo pendiente.\
**ID:** CU-18\
**Actor principal:** Usuario operativo.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El usuario ve lo que requiere su atención en sus clientes (vencimientos próximos, facturas vencidas, cobros por conciliar, envíos fallidos, aprobaciones, revisiones, incidencias y notificaciones) y accede directamente a resolverlo.

**Interesados:**
- **Usuario operativo:** quiere saber qué hacer hoy, también desde el móvil.
- **Administrador:** quiere una visión global para repartir el trabajo.

**Precondiciones:**
- Ninguna además de la común.

**Disparador:** El usuario empieza su jornada o recibe una notificación.

**Garantía de éxito:**
- El usuario ve solo lo de su ámbito, ordenado por prioridad y fecha.

**Garantía mínima:**
- Nunca se muestran datos de clientes que el usuario no puede ver.

**Escenario principal de éxito:**
1. El usuario consulta su trabajo pendiente.
2. El sistema reúne los pendientes de sus clientes y sus notificaciones, ordenados por prioridad y fecha.
3. El usuario elige un elemento.
4. El sistema le lleva a la acción que lo resuelve (por ejemplo, CU-15 para un cobro por conciliar).
5. El usuario marca la notificación como atendida.

**Extensiones:**
- **2a.** El usuario es administrador: el sistema incluye todos los clientes.
- **2b.** No hay nada pendiente: el sistema lo indica.

### CU-19: Consultar informes

**Caso de uso:** Consultar informes.\
**ID:** CU-19\
**Actor principal:** Usuario operativo.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El usuario obtiene informes filtrados (periodo, cliente, responsable, estado, moneda…) de los clientes que puede ver, desglosa sus cifras hasta las facturas o los cobros que las forman y los exporta a CSV o PDF.

**Interesados:**
- **Usuario operativo:** quiere listados de sus clientes que pueda exportar.
- **Dirección de Turbine:** quiere que nadie exporte lo que no puede ver.

**Precondiciones:**
- Ninguna además de la común.

**Disparador:** El usuario necesita un listado o revisar la situación de cobro.

**Garantía de éxito:**
- El informe contiene solo datos del ámbito del usuario y la exportación queda auditada.

**Garantía mínima:**
- Ninguna exportación incluye datos fuera de sus permisos.

**Escenario principal de éxito:**
1. El usuario elige un informe y sus filtros.
2. El sistema lo calcula con los datos que el usuario puede ver y lo muestra paginado.
3. El usuario desglosa una cifra.
4. El sistema muestra las facturas o los cobros que la forman.
5. El usuario exporta el resultado a CSV o PDF.
6. El sistema genera el fichero con la fecha, los filtros y el usuario, y registra la exportación.

**Extensiones:**
- **2a.** El informe es pesado: el sistema lo genera en segundo plano y avisa cuando está listo.
- **6a.** La exportación es masiva: el sistema la marca para que la revise un administrador.

### CU-20: Consultar dashboard financiero

**Caso de uso:** Consultar dashboard financiero.\
**ID:** CU-20\
**Actor principal:** Administrador.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El administrador consulta los indicadores financieros globales de Turbine (facturado, cobrado, pendiente y vencido por periodo, tiempo medio de cobro, clientes con más deuda e ingresos recurrentes esperados) y desglosa cada cifra hasta las facturas o los cobros que la forman.

**Interesados:**
- **Administrador:** quiere entender la situación financiera con cifras que pueda comprobar.
- **Dirección de Turbine:** quiere que ningún usuario operativo vea los datos financieros globales.

**Precondiciones:**
- El usuario es administrador.

**Disparador:** El administrador quiere revisar la situación financiera de Turbine.

**Garantía de éxito:**
- El administrador ve los indicadores del periodo y cada cifra cuadra con las facturas y los cobros que la forman.

**Garantía mínima:**
- Un usuario operativo nunca accede al dashboard financiero global.

**Escenario principal de éxito:**
1. El administrador consulta el dashboard financiero y elige el periodo.
2. El sistema calcula los indicadores con los datos de todos los clientes.
3. El sistema muestra facturado, cobrado, pendiente y vencido, tiempo medio de cobro, clientes con más deuda e ingresos recurrentes esperados.
4. El administrador desglosa una cifra.
5. El sistema muestra las facturas o los cobros que suman esa cifra.

**Extensiones:**
- **1a.** Un usuario operativo intenta acceder: el sistema lo rechaza, aunque use un enlace directo.
- **2a.** No hay datos en el periodo: el sistema muestra los indicadores a cero.
- **5a.** La cifra incluye facturas rectificadas: el desglose muestra la rectificativa junto a la factura original.

### CU-21: Gestionar usuarios

**Caso de uso:** Gestionar usuarios.\
**ID:** CU-21\
**Actor principal:** Administrador.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El administrador da de alta a los empleados de Turbine por invitación, solo con correos del dominio corporativo y sin autorregistro, les asigna un rol y los desactiva cuando dejan la empresa, conservando el rastro de lo que hicieron. El reparto de clientes entre empleados es un caso aparte (CU-22).

**Interesados:**
- **Dirección de Turbine:** quiere que solo accedan los empleados actuales y que quede rastro de quien se va.
- **Usuario operativo:** quiere tener su propia cuenta con los permisos de su rol.

**Precondiciones:**
- Ninguna además de la común.

**Disparador:** Alguien se incorpora a Turbine, cambia de rol o deja la empresa.

**Garantía de éxito:**
- La cuenta queda creada, modificada o desactivada con su rol, y el cambio queda auditado.

**Garantía mínima:**
- Ningún correo de fuera del dominio corporativo obtiene cuenta, y las acciones de un usuario desactivado conservan su autoría.

**Escenario principal de éxito:**
1. El administrador invita a un usuario con su correo y su rol.
2. El sistema comprueba que el correo es del dominio corporativo y que no existe ya.
3. El sistema crea el usuario y le envía una invitación que caduca.
4. El sistema registra el alta en la auditoría.

**Extensiones:**
- **1a.** El administrador cambia el rol de un usuario: el sistema aplica los nuevos permisos y registra el cambio.
- **1b.** El administrador desactiva a un usuario que deja Turbine: el sistema invalida sus sesiones activas, conserva su rastro y avisa de los clientes que se quedan sin responsable para reasignarlos (CU-22).
- **2a.** El correo no es del dominio autorizado o ya existe: el sistema rechaza el alta e indica el motivo.

### CU-22: Asignar clientes

**Caso de uso:** Asignar clientes.\
**ID:** CU-22\
**Actor principal:** Administrador.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El administrador decide qué empleado es responsable de cada cliente: reasigna clientes cuando alguien deja la empresa o cambia el reparto del trabajo, y delega temporalmente los clientes de quien se ausenta sin perder la asignación principal. El nuevo responsable hereda las tareas pendientes.

**Interesados:**
- **Usuario operativo:** quiere recibir sus clientes con el trabajo pendiente y poder cubrir a un compañero sin compartir contraseñas.
- **Dirección de Turbine:** quiere que ningún cliente se quede sin responsable y que el histórico siga indicando quién hizo cada acción.

**Precondiciones:**
- Ninguna además de la común.

**Disparador:** Cambia el reparto del trabajo: alguien entra, se va o se ausenta.

**Garantía de éxito:**
- Cada cliente afectado tiene su nuevo responsable, que hereda sus tareas pendientes, y el cambio queda auditado.

**Garantía mínima:**
- Solo un administrador cambia asignaciones, y el histórico conserva quién hizo cada acción antes del cambio.

**Escenario principal de éxito:**
1. El administrador elige uno o varios clientes.
2. El administrador indica el nuevo responsable.
3. El sistema muestra cuántos clientes cambian y qué tareas pendientes pasarán al nuevo responsable.
4. El administrador confirma.
5. El sistema cambia el responsable de cada cliente, le traspasa sus tareas pendientes y registra el cambio en la auditoría.
6. El sistema muestra el resultado, con los clientes reasignados y los que hayan dado error.

**Extensiones:**
- **1a.** Un usuario que no es administrador intenta cambiar una asignación: el sistema lo rechaza, aunque use un enlace directo.
- **2a.** El administrador delega los clientes durante una ausencia: indica el sustituto y las fechas; el sistema le da acceso temporal sin cambiar el responsable principal y lo retira solo al terminar el intervalo.
- **2b.** El nuevo responsable no tiene una cuenta activa: el sistema no permite la asignación.
- **5a.** Otro administrador ha cambiado a la vez el responsable de algún cliente: el sistema no sobrescribe ese cambio y lo indica en el resultado.

### CU-23: Configurar sistema

**Caso de uso:** Configurar sistema.\
**ID:** CU-23\
**Actor principal:** Superadministrador.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El superadministrador mantiene la configuración global: tarifas por rango, series, impuestos, plazos, cadencia de recordatorios, umbrales de aprobación, plantillas, datos fiscales de Turbine y calendario laboral. Los cambios se versionan, se auditan y nunca son retroactivos.

**Interesados:**
- **Dirección de Turbine:** quiere cambiar reglas sin tocar el código ni reescribir lo emitido.
- **Gestoría:** quiere que los datos fiscales y las series sean correctos.

**Precondiciones:**
- El usuario es superadministrador.

**Disparador:** Cambia una regla de negocio o la identidad corporativa, o hace falta una serie nueva.

**Garantía de éxito:**
- La nueva versión queda vigente desde su fecha de efecto, solo para operaciones nuevas.

**Garantía mínima:**
- Ninguna factura emitida cambia y una configuración inválida no se guarda.

**Escenario principal de éxito:**
1. El superadministrador elige el parámetro que quiere cambiar.
2. El sistema muestra el valor vigente y su historial.
3. El superadministrador introduce el nuevo valor y su fecha de efecto.
4. El sistema lo valida (por ejemplo, rangos sin solapes) y muestra una previsualización de su efecto.
5. El superadministrador confirma.
6. El sistema guarda la nueva versión y registra el cambio en la auditoría.

**Extensiones:**
- **4a.** El valor no es válido o una plantilla usa variables inexistentes: el sistema indica el error y vuelve al paso 3.
- **5a.** El superadministrador pide un envío de prueba de una plantilla: el sistema la envía solo a su propio correo.

### CU-24: Consultar auditoría

**Caso de uso:** Consultar auditoría.\
**ID:** CU-24\
**Actor principal:** Superadministrador.\
**Ámbito:** Sistema de facturación y cobros.\
**Nivel:** Objetivo de usuario.

**Descripción:** El superadministrador consulta y filtra el registro de auditoría para saber quién hizo cada acción relevante y cuándo: crear un cliente, cambiar un dato fiscal, emitir una factura, cambiar un vencimiento, aprobar un descuento, reasignar un cliente, marcar una disputa, conciliar un cobro a mano, cambiar la configuración o exportar datos. Nadie puede modificar ni borrar la auditoría, tampoco el superadministrador.

**Interesados:**
- **Superadministrador:** quiere responder rápido a «¿quién hizo esto y cuándo?».
- **Dirección de Turbine:** quiere que el pasado no se pueda reescribir y poder detectar exportaciones masivas de datos.

**Precondiciones:**
- El usuario es superadministrador.

**Disparador:** El superadministrador necesita saber quién hizo un cambio o revisar la actividad de un periodo.

**Garantía de éxito:**
- El superadministrador ve los registros que cumplen los filtros, con usuario, fecha y hora, acción, elemento afectado y, cuando lo hay, el valor anterior y el nuevo.

**Garantía mínima:**
- La consulta no modifica ningún registro y solo un superadministrador accede a ella.

**Escenario principal de éxito:**
1. El superadministrador consulta la auditoría.
2. El sistema muestra los registros más recientes, con usuario, fecha y hora, acción y elemento afectado.
3. El superadministrador filtra por usuario, tipo de acción, elemento (cliente, factura, configuración…) o periodo.
4. El sistema muestra los registros que cumplen los filtros.
5. El superadministrador abre un registro.
6. El sistema muestra su detalle: el valor anterior, el nuevo y, si la hubo, la aprobación asociada.

**Extensiones:**
- **1a.** Un usuario que no es superadministrador intenta acceder: el sistema lo rechaza, aunque use un enlace directo.
- **4a.** Ningún registro cumple los filtros: el sistema lo indica.
- **\*a.** Alguien intenta modificar o borrar un registro de auditoría: el sistema no lo permite y registra el intento.
