# Automatización de registro de pedidos con Power Automate Desktop

Proyecto RPA que procesa pedidos desde Excel y los registra en un portal web de demostración mediante Microsoft Edge. El flujo valida datos básicos, interpreta la respuesta del portal y guarda el resultado de cada pedido en el mismo archivo.

## Objetivo

Automatizar el traslado de información de Excel a un formulario web y mantener un registro del resultado, incluyendo pedidos registrados, referencias existentes y casos que requieren revisión.

## Herramientas

- Power Automate Desktop.
- Microsoft Excel de escritorio.
- Microsoft Edge con la extensión de Power Automate.
- Portal web de demostración.

## Funcionamiento

1. Abre Pedidos.xlsx y lee los datos usando la primera fila como encabezados.
2. Abre el portal en Microsoft Edge.
3. Recorre las filas y procesa únicamente las que tienen EstadoRPA igual a Pendiente.
4. Valida referencia, cliente, cantidad y precio unitario.
5. Si los datos superan las validaciones, limpia y completa el formulario web.
6. Envía el pedido y espera hasta 30 segundos el texto de confirmación.
7. Clasifica el resultado y actualiza Excel.
8. Guarda y cierra el archivo.
9. Muestra un resumen de la ejecución.

## Estructura de Excel

Se deben conservar los nombres y el orden de las columnas.

| Columna | Campo | Uso |
|---|---|---|
| A | Referencia | Identificador del pedido |
| B | FechaPedido | Fecha del pedido, almacenada como fecha de Excel |
| C | Cliente | Nombre del cliente |
| D | Distrito | Nombre de una opción del portal |
| E | CodigoProducto | Código del producto |
| F | Producto | Descripción del producto |
| G | Cantidad | Valor numérico mayor que cero |
| H | PrecioUnitario | Valor numérico mayor que cero |
| I | ImporteTotal | Importe calculado en Excel |
| J | EstadoRPA | Estado de procesamiento |
| K | CodigoPortal | Código obtenido del portal |
| L | Observacion | Respuesta del portal o motivo de revisión |
| M | FechaProcesamiento | Fecha y hora del procesamiento |

El flujo escribe los resultados en las columnas J, K, L y M.
La columna ImporteTotal no se utiliza para completar el formulario.

## Validaciones implementadas

- Referencia no vacía.
- Cliente no vacío.
- Cantidad no vacía, convertible a número y mayor que cero.
- Precio unitario no vacío, convertible a número y mayor que cero.

Si una fila presenta varios errores de validación, se conserva el último
motivo detectado según el orden de las comprobaciones.

## Estados

| Estado | Significado |
|---|---|
| Pendiente | Fila habilitada para procesamiento |
| Registrado | El portal confirmó un registro nuevo |
| Ya existente | El portal informó que la referencia ya estaba registrada |
| Revisar | Se detectaron datos inválidos, una respuesta no reconocida o un tiempo de espera agotado |

Las filas cuyo estado no sea Pendiente se omiten.

## Configuración y ejecución

1. Colocar Pedidos.xlsx en una carpeta local.
2. Configurar su ruta en la acción Iniciar Excel.
3. Mantener la hoja de pedidos activa antes de guardar el archivo.
4. Comprobar que los encabezados coincidan con los indicados.
5. Configurar la URL del portal y verificar los elementos web capturados.
6. Usar nombres de distrito y producto que coincidan con las opciones del portal.
7. Marcar los pedidos por procesar como Pendiente.
8. Guardar y cerrar Excel.
9. Ejecutar el flujo completo desde Power Automate Desktop.
10. Revisar el resumen y los resultados guardados en Excel.

La selección del producto utiliza este formato:

CodigoProducto - Producto

Ejemplo:

PROD-001 - Papel bond A4 75 g - paquete de 500 hojas

Durante la ejecución, evitar escribir o cambiar el foco de las ventanas,
ya que varias acciones utilizan el teclado físico.

## Corrección y reintento

Para un pedido con datos inválidos:

1. Leer el motivo en Observacion.
2. Corregir los datos.
3. Conservar la referencia original.
4. Cambiar EstadoRPA a Pendiente.
5. Guardar y cerrar Excel.
6. Ejecutar nuevamente.

Si se agotó el tiempo de espera, comprobar primero el portal:
el pedido podría haberse registrado aunque el robot no haya obtenido
la confirmación.

## Pruebas realizadas

| Caso | Resultado observado |
|---|---|
| Pedido nuevo con datos válidos | Registrado y código guardado |
| Referencia existente | Ya existente y código recuperado |
| Filas sin estado Pendiente | Omitidas, conservando sus resultados |
| Cantidad con texto abc | Revisar con motivo descriptivo |
| Precio unitario igual a cero | Revisar con motivo descriptivo |
| Dos pedidos inválidos seguidos de uno válido | 2 por revisar y 1 registrado |
| Corrección y reintento de los dos pedidos inválidos | 2 registros nuevos |
| Tiempo de espera provocado durante una prueba | Revisar, código vacío y observación |
| Ejecución posterior al tiempo de espera | Procesamiento normal recuperado |

Los contadores del resumen corresponden a la ejecución actual,
no al total histórico del archivo.

## Evidencias

- [Registro exitoso](Evidencias/01-registro-exitoso.png)
- [Pedido existente](Evidencias/02-pedido-existente.png)
- [Validación de datos](Evidencias/03-validacion-datos.png)
- [Resumen de validación](Evidencias/04-resumen-validacion.png)
- [Pedidos corregidos](Evidencias/05-pedidos-corregidos.png)
- [Tiempo de espera](Evidencias/06-tiempo-espera.png)

## Archivos del proyecto

- Pedidos.xlsx: archivo de trabajo.
- Flujo/RPA_Registro_Pedidos_Main.txt: acciones copiadas desde el diseñador.
- Evidencias/: capturas de las pruebas.
- Respaldo/: copia del Excel con resultados validados.
- Documentacion/: documentación complementaria.

El archivo de texto debe conservar su sección de elementos de interfaz.
No equivale a un paquete de solución exportado; al reutilizarlo se deben
verificar las acciones, los elementos web y la configuración del entorno.

## Alcance y limitaciones

- Proyecto probado en un portal de demostración.
- La detección de duplicados depende de la respuesta del portal.
- No incluye ejecución desatendida ni programación automática.
- No valida previamente la fecha ni la existencia de distrito y producto
  en el catálogo.
- No comprueba que la cantidad sea un número entero.
- Las comprobaciones de campos vacíos no incluyen una limpieza explícita
  de espacios.
- Maneja errores de conversión numérica y el tiempo de espera configurado.
  Otros errores de Excel o del navegador pueden detener la ejecución.
- El archivo se guarda al final del ciclo; no hay guardado por cada pedido.
- Los resultados anteriores de una fila se reemplazan al reprocesarla.
- La ruta del archivo y los elementos web requieren revisión al trasladar
  el flujo a otro equipo o portal.

## Autor

Juan Manuel Tafur Recuay