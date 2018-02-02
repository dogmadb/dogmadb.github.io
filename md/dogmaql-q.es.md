# DogmaQL/Q (queue)

*Tiempo de lectura: 5min*

**DogmaQL/Q** es la especificación de **DogmaQL** para servicio de mensajería basado en colas.

Un **servicio de mensajería** (*messaging service*) es un sistema de comunicación para el intercambio de mensajes entre dos o más componentes de software.

Sus principales usos son los siguientes:
- Implementación de chats.
- Notificación de información o datos en tiempo real como eventos, alertas, noticias, cambios de precio, registros, etc.
  Recibida la notificación, el cliente puede realizar una acción como, por ejemplo, su visualización o el envío de un correo.

Las principales características por las que se usa un servicio o sistema de mensajería son:
- Desacoplamiento de los componentes.
  Los emisores de los mensajes no se conectan a los receptores, sino al intermediario.
  De igual manera actúan los receptores, se conectan al intermediario.
  Por lo que los extremos no se comunican directamente, manteniéndose desacoplados.
- Escalabilidad del sistema si la carga crece o decrece.

## Modelos de mensajería

Generalmente, los servicios se implementan bajo uno de dos **modelos de mensajería** (*messaging models*), una manera de concebir el sistema.
Define los componentes que lo forman, cómo se conectan y cómo se intercambian los mensajes.
Estos modelos son el punto a punto y el basado en publicación/suscripción.

Bajo el **modelo punto a punto** (*point-to-point model*), los emisores envían sus mensajes a un intermediario, el cual se lo entregará a un único receptor.
Mientras que en el **modelo publicación/suscripción** (*pub/sub model*), el mensaje puede llegar a varios receptores.

Un **canal** (*channel*), **tema** (*topic*) o **cola** (*queue*) es un contenedor donde se publican mensajes.
Es el medio a través del cual los extremos intercambian mensajes.
El intermediario es el responsable de administrarlos.
Una vez los ha entregado, los suprime automáticamente.
Los emisores de los mensajes se los envían al intermediario indicándole a través de qué canales debe difundirlos.
A esta operación se la conoce formalmente como **publicación** (*publication*).
Mientras que cuando un componente desea recibir mensajes publicados por otros lo que hace es abonarse a los canales que son de su interés, lo que se conoce formalmente como **suscripción** (*subscription*).
Cada vez que el intermediario publica un mensaje en un canal, cuando pueda se lo enviará a sus suscriptores y, finalizado, lo suprimirá del canal.

No hace falta crearlos.
Se crean automáticamente cuando un cliente se suscribe al canal o bien cuando se publica un mensaje.
Y se destruyen con igual facilidad: cuando deja de tener mensajes y suscriptores.

## Sentencia insert

En **DogmaQL/Q**, se utiliza la sentencia `insert` para publicar mensajes en los canales.

Ejemplo:

```
insert in queue concerts(
  {
    artist = "Simple Minds"
    city = "Valencia"
    date = "Jun 29, 2018"
  }
  {
    artist = "Echo and the Bunnymen"
    city = "London"
    date = "Jun 1, 2018"
  }
)
```

Indicar la palabra clave `queue` después de `in`/`into`.
El canal o cola es `concerts`.
En el cual, se publican los dos mensajes indicados que, recordemos, se pasan en forma de objeto.

## Sentencia set

No disponible en **DogmaQL/Q**.

## Sentencia delete

No disponible en **DogmaQL/Q**.

## Sentencia update

No disponible en **DogmaQL/Q**.

## Sentencia select

No disponible en **DogmaQL/Q**.
