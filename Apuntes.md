# Apuntes AWS Serverless

Apuntes de como realizar la prueba 2 de aplicaciones, el paso a paso y demas.

## A Tener ojo

Trabajar siemrpre con el rol de LabRole, cada vez que pego código hago Deploy antes de probar. Si algo falla, miro los logs en la pestaña Monitorear, Ver registros en CloudWatch.

## 1. DynamoDB

1. En el buscador de arriba escribo DynamoDB y entro.
2. Clic en Tablas y después en Crear tabla.
3. Nombre de la tabla: (Nombre solicitado en la prueba)
4. Clave de partición: id_archivo,(lo mismo que arriba) tipo Cadena.
5. Dejo el resto por defecto y hago clic en Crear tabla.
6. Espero a que el estado diga Activa.

El nombre de la tabla tiene que ser igual al que va en el código de la Lambda.

## 2. Lambda

1. Busco Lambda y hago clic en Crear una función.
2. Dejo marcado Crear desde cero.
3. Nombre de la función: RRFunction.
4. Tiempo de ejecución: Node.js 24.x.
5. Abro Cambiar el rol de ejecución predeterminado, marco Utilizar un rol existente y elijo LabRole.
6. Clic en Crear función.
7. Bajo a Código fuente y abro el archivo index.mjs.
8. Modifico el archivo con el codigo entregado por el profe.
9. Cambio el nombre de la tabla del código por el mío (El que ingerse arriba).
10. Clic en Deploy. Tiene que salir un mensaje verde.
11. Si más adelante sale un error de que se acabó el tiempo (3 segundos), voy a Configuración, Configuración general, Editar y subo el tiempo de espera a 10 segundos.

## 3. S3

1. Busco S3 y hago clic en Crear bucket.
2. Nombre del bucket: uno único en minúsculas, por ejemplo rr-archivos-eval2-2026.
3. Región us-east-1. Dejo el resto por defecto y hago clic en Crear bucket.
4. Entro al bucket y abro la pestaña Propiedades.
5. Bajo hasta Notificaciones de eventos y hago clic en Crear notificación de eventos.
6. Nombre del evento: NotificarLambda.
7. En tipos de eventos marco Todos los eventos de creación de objetos.
8. En destino elijo Función Lambda y selecciono RRFunction.
9. Clic en Guardar cambios.
10. Vuelvo a la Lambda RRFunction y compruebo que en el diagrama aparezca S3 como desencadenador.


## 4. Probar que S3 guarda en DynamoDB

1. En el bucket abro la pestaña Objetos, clic en Cargar y Agregar archivos.
2. Elijo cualquier archivo pequeño y hago clic en Cargar.
3. Voy a DynamoDB, Tablas, RR, y hago clic en Explorar los elementos de la tabla.
4. Tiene que aparecer con el nombre del archivo, el peso, la fecha de subida.
5. Para mostrar el proceso, entro a la Lambda, Monitorear, Ver registros en CloudWatch, y abro el último flujo de registros. Tiene que decir que los metadatos se guardaron con éxito(opcional).

## 5. SNS

1. Busco Simple Notification Service y entro a Temas.
2. Clic en Crear tema.
3. Tipo Estándar y nombre RRTema. Clic en Crear tema.
4. Copio el ARN del tema, que aparece arriba en la página. Lo necesito para el código.
5. Clic en Crear suscripción.
6. Protocolo: Correo electrónico. Punto de enlace: mi correo. Clic en Crear suscripción.
7. Abro mi correo, busco el mensaje de AWS y hago clic en Confirm subscription. Si no llega, reviso spam.
8. Vuelvo a SNS y compruebo que la suscripción diga Confirmado.

Si no confirmo la suscripción no llega ningún correo, aunque lo demás esté bien.

## 6. Lambda con SNS

Opcional, ya esto es como la evaluacion formativa.

1. Entro a la Lambda RRFunction, pestaña Código, archivo index.mjs.
2. Selecciono todo, borro y pego el código de la función 2 que entrega el profe.
3. Cambio el nombre de la tabla por RR y el ARN del tema por el que copié.
4. Ahora tiene que llegar el correo y quedar el registro en DynamoDB.

## 7. CloudWatch, panel (dashboard)

1. Busco CloudWatch y entro a Paneles.
2. Clic en Crear panel, nombre Monitoreo.
3. Elijo el tipo de widget Línea y hago clic en Siguiente.
4. Elijo Métricas y Configurar.
5. Voy a Lambda, Por nombre de función, y marco Invocations, Errors y Duration de RRFunction.
6. Clic en Crear widget.
7. Arriba a la derecha hago clic en Guardar. El guardado automático viene desactivado, si no guardo se pierde el panel.

## 8. CloudWatch, alarma

1. En CloudWatch entro a Alarmas, Todas las alarmas y hago clic en Crear alarma.
2. Clic en Seleccione una métrica.
3. Voy a Lambda, Por nombre de función.
4. Marco solo una métrica, Errors de RRFunction. Si hay más de una marcada, el botón de abajo no se activa.
5. Hago clic en el botón de abajo para seleccionar la métrica.
6. Estadística: Suma. Período: 1 minuto.
7. Tipo de umbral: Estático. Condición: Mayor/Igual que 1.
8. Clic en Siguiente.
9. En Notificación, estado En alarma, elijo Seleccionar un tema de SNS existente y escojo RRTema.
10. Clic en Siguiente, nombre RRAlarmaErrores, Siguiente y Crear alarma.
11. Para probarla cambio el nombre de la tabla en el código de la Lambda por uno que no existe, hago Deploy y subo un archivo al bucket.
12. Espero dos o tres minutos hasta que la alarma pase a En alarma y llegue el correo.
13. Dejo el nombre de la tabla bien otra vez y hago Deploy.


## 9. Memoria y simultaneidad aprovisionada

Memoria a 512 MB:

1. Entro a la Lambda, Configuración, Configuración general y hago clic en Editar.
2. En Memoria pongo 512.
3. Clic en Guardar.

Simultaneidad aprovisionada en 2:

Esta parte me dio error por los permisos, nose si el profe lo va a solicitar en la prueba.

1. Abro la pestaña Versiones y hago clic en Publicar nueva versión.
2. Pongo una descripción, por ejemplo Probar, y hago clic en Publicar. No se puede configurar sobre $LATEST, tiene que ser una versión.
3. Voy a Configuración, Simultaneidad y detección de recursividad, y busco Simultaneidad aprovisionada.
4. Clic en Agregar o Editar.
5. Tipo de calificador: Versión. Versión: 1.
6. Simultaneidad aprovisionada: 2.
7. Clic en Guardar.


## 10. API Gateway

Primero creo la tabla y la Lambda, después la API.

Tabla:

1. En DynamoDB, Crear tabla.
2. Nombre ItemsTable, clave de partición ItemId (Cadena). Clic en Crear tabla.

Lambda:

1. En Lambda, Crear una función, Crear desde cero.
2. Nombre ItemsFunction, Node.js 24.x, rol existente LabRole.
3. Pego el código que entrega el profe en index.mjs y hago Deploy.

API:

1. Busco API Gateway.
2. En la tarjeta API HTTP hago clic en Crear. No en Importar.
3. Clic en Agregar integración y elijo Lambda.
4. En Función de Lambda elijo ItemsFunction.
5. Nombre de la API: ItemsAPI. Clic en Siguiente.
6. En Configurar rutas, el asistente trae una ruta por defecto que hay que cambiar. Dejo POST /items, GET /items y DELETE /items, las tres con la integración ItemsFunction.
7. Clic en Siguiente.
8. En Etapas dejo $default con la implementación automática activada. Clic en Siguiente.
9. Reviso las tres rutas y hago clic en Crear.
10. En la página de la API copio la URL de invocación.

En la API HTTP no hay botón Deploy manual, la implementación automática lo hace sola.

Probar:

1. Pego la URL de invocación en el navegador y le agrego /items?id=1. Para que muestre algo, el ítem con ItemId 1 tiene que existir en la tabla.
2. Si responde con los datos del ítem, la API, la Lambda y DynamoDB están conectadas.
3. El navegador solo hace GET. Para POST y DELETE uso curl o Postman, o la pestaña Probar de la Lambda si el profe lo permite.

Versión con S3:

Esto va segun lo solicitado en la prueba.

1. Creo un bucket con nombre único.
2. Creo la Lambda S3FileManager (Node.js 24.x, LabRole) con el código del profe y cambio el nombre del bucket en el código.
3. Creo la API FileManagementAPI con las rutas POST, GET y DELETE en /files.
4. Pruebo en el navegador con la URL de invocación y /files, que lista los archivos del bucket.

## 11. EventBridge

Antes de crear la regla:

1. Voy a S3, mi bucket, pestaña Propiedades.
2. Bajo hasta Amazon EventBridge y hago clic en Editar.
3. Elijo Activado y hago clic en Guardar cambios. Sin esto, S3 no manda eventos a EventBridge.
4. Creo una Lambda de destino, por ejemplo RREventBridgeFunction (Node.js 24.x, rol LabRole), con el código del profe, y hago Deploy.

Crear la regla:

1. Busco EventBridge, entro a Buses de eventos y hago clic en el bus default.
2. Busco Reglas y hago clic en Crear regla.
3. Paso 1, Definición de detalles de reglas: nombre RRReglaS3, bus de eventos el predeterminado, tipo de regla Regla con un patrón de eventos. Siguiente.
4. Paso 2, Crear patrón de eventos: en Origen del evento dejo Eventos de AWS o eventos de socios de EventBridge.
5. En Método de creación dejo Formulario del patrón de eventos.
6. En el desplegable Servicio de AWS elijo Simple Storage Service (S3).
7. Tipo de evento: Amazon S3 Event Notification. Eventos específicos: Object Created. Bucket específico: elijo el mío por nombre.
8. Reviso que el cuadro Patrón de eventos de la derecha se haya llenado con el JSON. Siguiente.
9. Paso 3, Seleccionar destinos: en Seleccione un destino elijo Función Lambda. Ubicación de destino: Destino en esta cuenta. Función: RREventBridgeFunction.
10. En Permisos dejo marcado Usar el rol de ejecución y en Rol de ejecución elijo Usar otro rol. En la lista elijo LabRole.
11. Dejo Configuración adicional cerrado. Siguiente.
12. Paso 4, Configurar etiquetas: no agrego nada. Siguiente.
13. Paso 5, Revisar y crear: reviso el nombre, el patrón y el destino. Clic en Crear regla.
14. En la página de la regla compruebo que diga Habilitada y que en Destinos aparezca mi Lambda.

Si LabRole da error, vuelvo al paso 3 y desmarco Usar el rol de ejecución. Para Lambda, EventBridge usa entonces el permiso de la propia función. Lo compruebo en la Lambda, Configuración, Permisos, Política basada en recursos, donde tiene que aparecer events.amazonaws.com.

Probar la regla:

1. Voy al bucket, Cargar, y subo un archivo.
2. Entro a la Lambda RREventBridgeFunction, Monitorear, Ver registros en CloudWatch.
3. Abro el último flujo de registros. Si todavía no aparece, espero unos 30 segundos y recargo.
4. Tiene que aparecer el evento con el tipo Object Created, el nombre del bucket y el archivo.

Si no se dispara, reviso que EventBridge esté activado en las propiedades del bucket, que el nombre del bucket del patrón sea el correcto y que la Lambda tenga hecho el Deploy.

Si piden un evento propio en vez de uno de S3, en el paso 4 elijo Otros y Patrón personalizado (editor JSON) y escribo el patrón que pidan.

