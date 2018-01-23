# Series temporales

*Tiempo de lectura: 15min*

**DogmaQL/TS** es la especificación de **DogmaQL** para motores de series temporales.

Una **serie temporal** (*time series*) es una sucesión de objetos de datos que, como aspecto principal, se generan en determinados momentos del día.
Sus datos se organizan a partir del momento del día que los generó, el cual se conoce formalmente como **tiempo** (*time*) o **marca temporal** (*timestamp*).
Cada objeto representa un registro de datos y se conoce formalmente como **muestra** (*sample*).

**DogmaQL** proporciona la especificación **DogmaQL/TS** para trabajar con tablas que almacenan series temporales.

Una **base de datos de series temporales** (*time-series database*) es aquella que optimiza el almacenamiento y la consulta de datos basados en tiempo.
Como todo modelo de datos, este tipo de base de datos establece cómo se almacenan y acceden los datos por parte de los usuarios.
Debido al uso de los datos, generalmente se cumple lo siguiente:

- Las escrituras suelen ser operaciones de inserción, no siendo frecuentes las de actualización.
  Es más, generalmente la operación de actualización no suele soportarse.
  Las inserciones suelen ser frecuentes.
- Las lecturas suelen ser también muy frecuentes y consisten en buscar datos a partir de la clave principal o intervalos de tiempo.
- Las supresiones no suelen ser frecuentes, pero se soportan para purgar datos expirados que ya no son útiles.

Las series temporales son muy utilizadas hoy en día.
Más de lo que nos podemos imaginar.
Principalmente en:

- Estadística.
- Reconocimiento o revelación de patrones o tendencias de precios, acciones, eventos, enfermedades, contaminación, tráfico, etc.
- Pronósticos del tiempo, precios, enfermedades, contaminación, tráfico, etc.
- Monitorización y seguimiento de precios, ventas, acciones, eventos, visualizaciones, mensajes enviados a intervalos de tiempo, etc.

También se utiliza en distintos sectores como, por ejemplo:

- Banca.
- Ciencia.
- Comercio.
- Gobierno.
- IoT.
- Robótica.
- Sanidad.
- Telecomunicaciones.

Entre las organizaciones que almacenan y utilizan bases de datos de series temporales encontramos:
`Airbnb`, `Alibaba`, `bet365`, `Cisco`, `eBay`, `Facebook`, `IBM`, `Mozilla`, `Nasa`, `Netflix`, `OVH`, `PayPal`, `Telefónica`, `The Weather Company`, `Twitter`, `Wikipedia Foundation` y `Xiaomi`.

## Tablas de series temporales

Una **tabla de serie temporal** (*time-series table*) es aquella que almacena objetos o muestras relacionadas con algún aspecto particular.
Por ejemplo, supongamos que necesitamos almacenar un registro con la temperatura media de cada día de las poblaciones más importantes de España.
Esto se puede hacer mediante una tabla de series temporales, donde la serie hace referencia a la temperatura media a lo largo de cada día en cada población.
También podríamos almacenar la evolución de los precios de acciones, del tráfico de una ciudad, de enfermedades, de los niveles de CO2, etc.

En **DogmaQL**, las tablas de este tipo se encuentran en el espacio de nombres `ts`.
Cualquier tabla almacenada en este espacio de nombres será una tabla de series temporales.

### Tipos de campos

Una tabla de series temporales presenta los siguientes tipos de campos:

- Un **campo de tiempo** (*time field*), el que tiene como nombre `time`.
  Indica el momento del día al que hace referencia el registro o muestra.
- El **campo clave** (*key field*).
  Identifica al propietario del registro o muestra como, por ejemplo, un nombre de ciudad, un identificador de sensor, etc.
- Uno o más **campos de medida** (*measure fields*).
  Campos numéricos que contienen valores o tomas de algún aspecto del registro.
  Como, por ejemplo, el nivel de CPU, la temperatura, la humedad, los productos vendidos, la cotización de la acción, etc.
- Cero, uno o más campos adicionales que poporcionan información suplementaria del registro.

En las tablas de series temporales, las claves principales son compuestas y están formadas por el campo clave y el campo de tiempo.
En el momento de crear la tabla, sólo hay que indicar el campo clave, porque el de tiempo, `time`, queda implícito.
Así pues, nunca podremos tener dos o más muestras para el mismo campo clave y la misma marca de tiempo.

Veamos un ejemplo de inserción de muestra:

```
insert into ts.weather({
  city = "valencia"
  time = 2018011210   #12 de enero de 2018 a las 10 horas
  temp = 8
  hum = 63
})
```

El campo `city` es el campo clave.
`time`, el campo de tiempo.
Y `temp` (temperatura) y `hum` (humedad), dos campos de medida.
Observe que una misma muestra puede disponer de tantos campos de medida como sea necesario, relacionados con lo que se está registrando en la tabla y siempre de tipo numérico.
Haciendo, además, referencia al mismo momento en el que se tomaron.

### Cuanto

El **cuanto** (*quantum*) representa la frecuencia a la que se generan los registros o muestras de datos.
Como, por ejemplo, cada hora, cada 15 minutos, cada 3 días, etc.

Generalmente, todas las muestras de una tabla presentan el mismo cuanto.

### Creación de tablas de series temporales

He aquí un ejemplo de creación de este tipo de tabla:

```
insert into db.tables({
  table = "ts.temps"
  key = "city"
})
```

Aunque la clave principal es compuesta, sólo hay que indicar el campo clave, pues el campo de tiempo queda implícito.

## Campo de tiempo

Recordemos que todo objeto o muestra debe presentar el campo `time`, el cual contiene el momento del día al que hace referencia el registro.
Recordemos también que el cuanto es la frecuencia con la que se genera el registro, el cual, en nuestro caso, puede representarse en cualquiera de los siguientes formatos numéricos:

- `yyyy`, se utiliza en tablas donde el cuanto es el año.
  Ejemplo: `2018`.
- `yyyymm`, se usa cuando el cuanto hace referencia a mes y año.
  Ejemplo: `201802`.
- `yyyymmdd`, éste a año, mes y día.
  Ejemplo: `20180201` (1 de febrero de 2018).
- `yyymmddhh`, este otro a año, mes, día y hora.
  Ejemplo: `2018020119` (01 de febrero de 2018 a las 7 de la tarde).
- `yyyymmddhhmm`, a año, mes, día, hora y minuto.
  Ejemplo: `201802011935`.
- `yyyymmddhhmmss`, el más completo haciendo referencia hasta del segundo.
  Ejemplo: `20180201193542`.

Cuando insertamos en una tabla, aunque no es obligatorio, es muy recomendable que todos los registros tengan el mismo formato de cuanto.

## Sentencia insert

Mediante un `insert` de **DogmaQL**, se puede insertar uno o más registros en una tabla de series temporales.
No olvidemos indicar el espacio de nombres `ts`, indicando así que estamos trabajando con una tabla de series temporales.
Ejemplo:

```
insert into ts.weather(
  {city="Valencia", time=2018011001, temp=4}
  {city="Valencia", time=2018011002, temp=4}
  {city="Valencia", time=2018011003, temp=5}
  ...
)
```

**Nota importante**. El primer campo siempre debe ser el campo clave; y el segundo, el campo de tiempo.
Los restantes es indiferente.
Por favor, no lo olvide.

## Sentencia set

Es similar al `insert`, pero si ya existe una muestra para la clave principal indicada, la reemplaza.
Ejemplo:

```
set into ts.co2(
  {station="Valencia, Avd. Francia", time=2018011210, aqi=28, o3=5, no2=28, so2=4, pm25=25, pm10=12}
  {station="Valencia, Politècnic", time=2018011210, aqi=50, 03=0, no2=32, so2=2, pm25=50, pm10=22}
  {station="Valencia, Molí del Sol", time=2018011210, aqi=53, o3=2, no2=31, so2=3, pm25=53, pm10=14}
  ...
)
```

**Nota importante**. Al igual que en la sentencia `insert`, el primer campo de cada objeto debe ser el campo clave; y el segundo, `time`.

## Sentencia update

Las bases de datos de series temporales no suelen permitir el uso de la sentencia `update` para modificar campos individuales de los registros.
A lo sumo, se soporta el `set`, permitiendo así modificar todo el dato.
Pero ojo, algunos motores no soportan ni siquiera esta sentencia `set`.

Tampoco es posible modificar los metadatos de una tabla de series temporales mediante un `update`.

## Sentencia delete

Para suprimir muestras, hay que utilizar la sentencia `delete`, indicando en la cláusula `where` las muestras a suprimir.

No se puede usar la cláusula `if` de **DogmaQL**. Y sí, la cláusula `return old`.

Ejemplo:

```
delete from ts.quotes
where quote == "Apple Inc"
return old
```

## Sentencia select

Para acceder a datos, hay que utilizar la sentencia `select` de **DogmaQL** con una cláusula `where` que indique las muestras a acceder.
A continuación, se muestra un ejemplo con el que acceder a los registros de los últimos 7 días:

```
select from ts.visits
where hospital == "Valencia, La Fe" and time == "last 7d"
```

A diferencia de la sentencia `delete`, se puede omitir la cláusula `where`.
En cuyo caso, se accederá a todos los registros de la tabla.
Ejemplo:

```
select from ts.visits
```

Las cláusulas `if` y `set` no se pueden usar en tablas de series temporales.

## Comparación de acceso a registro/muestra/objeto

Las bases de datos optimizadas para el almacenamiento de series temporales generalmente sólo permiten el acceso a los datos mediante el campo de tiempo y/o la clave.
Ningún otro es posible, pues en principio no tiene sentido.
Cuando las muestras se guardan en almacenes de este tipo, es porque se accederán mediante estos campos y no otros.

Cuando se usa el campo clave, sólo se permite la comparación por igualdad (`==`).

Mientras que el campo `time` se puede comparar como sigue:

- `time == valor`
- `time < valor`
- `time <= valor`
- `time > valor`
- `time >= valor`
- `time between inicio and fin`

Observe que se suele indicar un momento concreto o un intervalo de tiempo.
Es más, y tal como veremos en breve, los motores de bases de datos de series temporales suelen representar determinados períodos de tiempo muy fácilmente.
Como, por ejemplo, el acceso a los datos de los últimos *X* días.

Vamos a ver unos ejemplos ilustrativos:

```
#suprime todas las muestras de Valencia
delete from ts.weather
where city == "Valencia"

#suprime todas las muestras de un determinado momento
delete from ts.weather
where time == 2018011202

#suprime todas las muestras entre dos momentos
delete from ts.weather
where time between 20170101 and 20171231

#suprime todas las muestras de Valencia en un rango
delete from ts.weather
where city == "Valencia" and time between 20170101 and 20171231
```

A la hora de determinar las muestras a partir de una comparación numérica, el valor indicado es redondeado.
Supongamos que estamos registrando las temperaturas y tenemos una muestra para cada hora.
Esto significa que tendremos el campo `time` con el siguiente formato: `yyyymmddhh`.
Veamos algunos ejemplos de uso teniendo en cuenta el formato usado:

- `time == 2018`, hace referencia a todos los registros del año 2018, independientemente del mes, del día y de la hora.
- `time == 201801`, en este caso, a todos los registros del mes de enero de 2018, sin importar el día y la hora.
- `time == 20180102`, ahora, a todos los registros del día 2 de enero de 2018, sin importar la hora.
- `time == 2018010203`, con este formato, al registro del día 2 de enero de 2018 a las 3 horas.

Observe que durante la búsqueda, el formato se completa a un intervalo.
El incio se redondea al inicio del año, mes, día, hora...
Mientras que el final del intervalo al final del año, mes, día, hora...
Sagún corresponda.
Así pues, `time == 2018` es lo mismo que `time between 20180101000000 and 20181231235959`.
Y `time == 201801` es similar a `time between 20180101000000 and 20180131235959`.

El campo `time` se puede comparar con un valor numérico que indique un momento dado o un valor textual:

- `today`, representa al día de hoy.
- `yesterday`, representa al día de ayer.
- `last Xm`, hace referencia a los últimos *X* minutos.
  Ejemplo: `last 60m`.
- `last Xh`, hace refencia a las últimas *X* horas, omitiendo la actual.
  También es posible `last Xd`, `last Xw`, `last XM` y `last Xy`.
- `[last Xh]`, representa las últimas *X* horas, incluyendo la actual.
  También es posible `[last Xd]`, `[last Xw]`, `[last XM]`, `[last Xy]`.
- `Xm ago`, hace referencia a *X* minutos atrás.
  También es posible `Xh ago`, `Xd ago`, `Xw ago`, `XM ago` y `Xy ago`.

El uso de los formatos `last XhdwMy` y `[last XhdwMy]` se ilustra mejor con unos ejemplos.
Supongamos que estamos a 3 de enero de 2018 y deseamos acceder a los datos de los últimos 3 meses.
La pregunta a responder es: ¿queremos el último trimestre del año que acabamos de dejar, sin importar los 3 días que llevamos del actual o bien los últimos tres meses, esto es, desde el 3 de octubre al 3 de enero?
Si la respuesta es del 1 de octubre al 31 de diciembre, usaremos `last 3M`.
En cambio, si es del 3 de octubre al 3 de enero, usaremos `[last 3M]`.

Otros ejemplos ilustrativos interesantes:

```
#desde hace 30 días y ayer
select from ts.weather
where time between "30d ago" and "yesterday"

#los de hace 180 días
select from ts.weather
where time == "180d ago"

#de los últimos 180 días
select from ts.weather
where time == "[last 180d]"
```

### Cláusula where

Como hemos indicado, para determinar el acceso a los datos, ya sea en el `where` de un `delete` o de un `select`, sólo es posible indicar los campos de la clave principal, esto es, el campo clave y/o el campo de tiempo. Es importante recordar una cosa, en caso de indicarse los dos, primero indicar el campo clave y después el tiempo.

Y de nuevo, no olvidemos que el campo clave sólo se puede comparar por igualdad (`==`) y el campo de tiempo mediante los siguientes operadores:
`==`, `<`, `<=`, `>`, `>=` y `between`.
