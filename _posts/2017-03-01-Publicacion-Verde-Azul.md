---
layout: post
title: Ventajas de Sistema de Publicación Azul - Verde (Blue - Green)
date: 2017-03-01 21:47:07.000000000 +02:00
type: post
published: true
status: publish
categories:
- Deployment
- Devops
tags: []
---

En nuestro proyecto actual, hemos montado un sistema de publicación basado en el enfoque [azul - verde propuesto por Martin Fowler][1].

## Concepto y Objetivo

El concepto es bastante sencillo, a la hora de publicar:
1. En todo momento hay un servidor activo y otro inactivo. Estar activo quiere decir que los usuarios que se conectan van a esa dirección. 
2. Cuando publicas una nueva versión del proyecto lo publicas en el servidor inactivo.
3. Realizas las comprobaciones que necesites en producción de la nueva versión (incluso puedes lanzar test en producción)
4. Nosotros normalmente realizamos un par de acciones para llenas las cachés de consultas.
5. Activamos la nueva versión que supone redirigir el tráfico al nuevo servidor.

La redirección es transparente al usuario ya que hay un enrutador que desvia la url principal a cada uno de los servidores según un test de salud.

El objetivo principal por el que nos planteamos e implementamos este enfoque fue: **Poder realizar actualizaciones con tiempo de desconexión (downtime) 0**.

Nosotros aplicamos un sistema de integración contínua y publicamos varias veces al día, al principio cuando no teníamos este enfoque, cada vez que publicábamos una nueva versión del proyecto suponía un corte de 1-2 minutos en la aplicación, entre que se copiaban los archivos, se reiniciaba el servidor y volvía estar activo el servicio web. Esto era incompatible (o como mínimo muy molesto) con nuestros usuarios que utilizan la aplicación como aplicación de trabajo de forma ininterrumpida e intensiva de 8:00 a 23:00. Además tenía un efecto colateral peligroso, si estabas en medio de un proceso de inserción de datos y se actualizaba el servidor, se recargaba la página y se perdía toda la información introducida, como os podéis imaginar los usuarios estaban contentísimos cuando hacíamos una actualización y les pillaba en medio de inserciones o actualizaciones masivas (haciendo amigos a diario).

Estos problemas nos obligaban a realizar las actualizaciones en la ventana de 23:00 a 8:00, pero no era muy operativo, ya que en los primeros pasos de la aplicación había cambios que era necesario hacer lo antes posible (cagadas importantes) y al actualiar las versiones en momentos en los que no había apenas usuarios, si aparecían errores o problemas de rendimiento, los sufríamos a las 8:00 (así sin café ni nada) y teníamos que volver a actualizar a versiones anteriores con la exasperación contínua del usuario. 

## Implementación y Ventajas

Siguiendo [este artículo][2] la puesta en marcha no fue complicada, y la verdad es que es una de las mejoras cosas que hemos podido hacer. A parte de cumplir nuestro objetivo original hemos vista que hay muchos proceso que podemos hacer y resolver gracias a nuestro sisitema de publicación.

Algunas de estas ventajas:
1. Si una publicación va mal, y se pone en producción (el efecto de 5 segundo después de publicar que suene los tres teléfonos de soporte a la vez...) en 1 segundo vuelves a poner la versión anterior en producción y todos tan contentos.
2. En el caso anterior puedes probar qué ha fallado con los datos y versión que ha fallado en producción, valor incalculable para esos problemas que tiene que ver con los nuevos datos.
3. Puedes elegir entre avisar al usuario para que refresque (completamente necesario si hay cambio en el front end) o hacerlo de forma transparente si los cambios son solo en la parte servidor.
4. Puedes poner versiones experimentales en producción e indicar a un usuario o usuarios clave que se conecten a la dirección inactiva para validar los cambios o el nuevo aspecto.

A nosotros nos costó un poco meternos en faena y montarlo pero ahora no podríamos vivir sin él. Dí adios a las publicaciones a las 3:00 am, ni a cruzar los dedos para que a las 7:30 o 8:00 no vaya nada mal. Ahora publicamos a cualquier hora y si la cosa no va bien volvemos a la versión anterior y evaluamos impacto.

Todo esto asegurando que el trabajo diario sale sin problemas y sin verse afectado por nuestra política de actualizaciones. Los usuarios ven mucho antes las mejoras y sufren menos, lo que supone usuarios más contentos, el equipo más confiado y más productividad en todos los campos. Vamos cada línea de código y minuto empleado en montarlo tiene un retorno inmediato y continuo espectacular. 

[1]: https://martinfowler.com/bliki/BlueGreenDeployment.html
[2]: https://kevinareed.com/2015/11/07/how-to-deploy-anything-in-iis-with-zero-downtime-on-a-single-server/