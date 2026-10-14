# Historias de usuario - Especificación del producto

Este documento resume la visión funcional de Manager Library y sirve como base para que cada capa del proyecto implemente las historias del negocio sin perder consistencia con la especificación.

## HU-01: Iniciar sesión

Como bibliotecario, quiero iniciar sesión para acceder al sistema y realizar operaciones autorizadas.

## HU-02: Crear bibliotecarios

Como administrador, quiero dar de alta bibliotecarios para organizar la atención de la biblioteca.

## HU-03: Gestionar libros

Como administrador, quiero administrar el catálogo para registrar, editar y dar de baja libros correctamente.

## HU-04: Buscar libros

Como bibliotecario, quiero buscar libros por texto y categoría para atender rápidamente las solicitudes de consulta.

## HU-05: Gestionar socios

Como bibliotecario, quiero gestionar socios para registrar los usuarios que pueden retirar materiales.

## HU-06: Registrar préstamo

Como bibliotecario, quiero registrar un préstamo cuando haya ejemplares disponibles para entregar materiales a los socios.

## HU-07: Registrar devolución

Como bibliotecario, quiero registrar la devolución para cerrar el movimiento y actualizar el inventario.

## HU-08: Consultar préstamos

Como bibliotecario, quiero ver un préstamo y listarlos para revisar el historial y el estado de cada operación.

## HU-09: Identificar vencidos

Como bibliotecario, quiero detectar préstamos vencidos para tomar acciones de recordatorio o seguimiento.

## HU-10: Reporte de más prestados

Como administrador, quiero consultar el reporte de los libros más prestados para evaluar tendencia y compra.

## Reglas de negocio transversales

- El sistema debe respetar roles de bibliotecario y administrador.
- Los préstamos dependen de la disponibilidad de ejemplares.
- Las devoluciones duplicadas deben rechazarse.
- Los reportes deben conservar el título histórico de un libro aunque cambie de nombre.
- Se aplica baja lógica en lugar de eliminación física para conservar trazabilidad.
