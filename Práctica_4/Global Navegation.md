# Navegación global  
  
## Objetivo de la práctica  
El objetivo de esta práctica es programar un coche que haga de taxi autónomo, donde el usuario marca un punto en el mapa y el coche debe planificar por sí solo el recorrido más rápido hacia el punto seleccionado por el usuario y conducir hasta allí. Al llegar al destino seleccionado, el coche debe pararse y esperar al siguiente destino.  
La planificación debe hacerse usando el algoritmo **Gradient Path Planning (GPP)**. Este algoritmo construye un campo escalar sobre el mapa que es la suma de dos subcampos, uno de propagación hasta el destino y otro de penalización por cercanía a obstáculos. El coche consulta ese campo en cada iteración para saber hacia dónde moverse y gobierna su velocidad lineal (V) y su velocidad angular (W) con un bucle reactivo.  

## Planteamiento general de la práctica  
Esta práctica la separé en dos bloques:  
* Planificación (una vez por destino): Construir el campo.
* Pilotaje (en cada iteración): Bajar por el campo para obtener una ruta local y convertirla en velocidad lineal (V) y angular (W).
El campo no es una ruta fija; es un plan que se puede consultar en cualquier momento. Da igual dónde esté el coche, ya que basta con mirar su celda y bajar por el campo para saber hacia dónde ir.
  
## Construcción del campo  
### Mapa y coordenadas  
   El mapa es una imagen de 400x400 celdas, donde cada celda equivale a 1'25m (500 metros de ciudad entre 400 celdas) y los píxeles con valor 255 son carretera.  
   `WebGUI.worldToGrid` devuelve (columna, fila), pero la matriz se registra como [fila][columna], por lo que creé una función `mundo_a_celda` que diera la vuelta al par. Además, la fila del mapa corresponde al eje x del mundo y la columna al eje y, por lo que el ángulo hacia un punto de rejilla se calcula como `atan2(Δcolumna, Δfila)`.  
### Engordado de obstáculos  
Antes de empezar con el movimiento del coche, hay que agrandar los obstáculos 4 celdas (5 metros) con una dilatación de kernel elíptico. Las celdas que quedan dentro de esa franja se consideran prohibidas; la onda no entra en ellas y el coche no puede planificar por esas zonas. Así, el coche se trata como un punto y el margen de seguridad, radio del coche más un extra, queda incluido en el mapa.  

### Subcampo de penalización  
Usando `cv2.distanceTransform`calculo la distancia de cada celda libre a la zona prohibida. La penalización vale:  
`penalización = máx(0, 12 - distancia) * 0'7`  
Es decir, se sube al acercarse a una pared y es cero a partir de 12 celdas. Solo depende del mapa, así que se calcula una vez al arrancar. Mantuve la pendiente (0'7) por debajo de 1 para que el campo siga bajando hacia el destino sin crear mínimos locales.  

<img width="350" height="327" alt="Captura de pantalla 2026-11-25 a las 23 16 38" src="https://github.com/user-attachments/assets/21a07fc0-0bda-4adb-a720-ea69ae3da16d" />
  

### Subcampo de propagación  
Es una onda que sale del destino, con valor 0, y se extiende por las calles libres con el algoritmo de **Dijkstra**, el cual es una cola de prioridad que expande siempre la celda con menor coste. Se consideran los 8 vecinos, con coste 1 en horizontal y vertical y 1'4 (√2) en diagonal. Si una celda se alcanza por un camino más barato, se actualiza. Las celdas prohibidas no se llegan a expandir.  
La onda se propaga por todo el mapa alcanzable (no se detiene al llegar al coche), de modo que hay valor del campo en cualquier punto por el que pueda pasar el coche.  

### Campo total  
El campo final es la suma de los dos subcampos: `campo total = propagación + penalización`.  
  
<img width="333" height="330" alt="Captura de pantalla 2026-11-25 a las 23 23 04" src="https://github.com/user-attachments/assets/49654ff6-3d91-4036-918d-f3b4f18ca223" />
  
Si el usuario pica sobre un obstáculo, fuera del mapa o dentro de la zona engordada, el destino se descarta con un aviso. Cuando cambia el destino, se recalcula el campo.  

## Pilotaje del campo a V y W  
### Ruta local y punto de mira  
En cada iteración se localiza la celda del coche y se baja por el campo 36 celdas por delante (45 metros) para obtener una ruta local. En cada paso no elijo simplemente el vecino de menor valor, sino el que ofrece la mayor bajada por unidad de longitud; así consigo seguir mejor la dirección del gradiente. La ruta se suaviza con una media móvil de 7 puntos para quitar el zigzag de la rejilla.  
El **punto de mira** es el punto de esa ruta situado a una distancia de `min(20, 6 + 0'7*V)` metros, es decir, el coche mira más lejos cuanto más rápido va. Para elegirlo, los puntos de la ruta se apartan de las paredes cercanas (hasta 3 celdas, moviéndolos hacia donde crece la distancia a la pared). Esto no cambia el campo, el cual sigue siendo la suma de los dos subcampos, solo desplaza los puntos de la ruta local para que el coche abra la curva antes de llegar a la esquina y salga recto de ella.  

### Velocidad angular  
El error de ángulo es la dirección hacia el punto de mira menos el `yaw` del coche, normalizado entre -π y π. W sale de un control proporcional-derivativo:  
`W = Kp * error + Kd * (error - error_anterior)`, con Kp = 1 y Kd = 0'4.  
Con tres añadidos. La ganancia baja cuando V supera 15, la salida se satura en +-2'5 y se mezcla con la W anterior para que el giro sea más suave.  

### Velocidad lineal  
Para cada punto de la ruta local, calculo cuánto puede ir el coche por la curva y por el espacio.  
+ Por la curva: `v = √(A_lat / curvatura)`, con A_lat = 6'5.
+ Por el espacio: `v = 5 + 4 * holgura`, donde la holgura es la distancia (en celdas) a la zona prohibida. En una calle estrecha o pegado a una esquina, el límite baja.
Después, tengo en cuenta el **frenado anticipado**, en el cual la velocidad permitida ahora es la que deja llegar a cada punto con una deceleración de 6, es decir, `√(v_punto^2 + 2*6*s)`, con s la distancia hasta ese punto. Si el destino está a la vista, se frena para llegar con velocidad 2. Por último, V se reduce un poco más si el coche está mal orientado (`V / (1 + 0'8*|error|)`) y solo puede subir 0'3 por iteración, para tener un arranque progresivo. La velocidad máxima en recta es de 25.

### Vuelta del coche sobre sí mismo  
Si el error de ángulo supera 1'2 radianes, el coche se para y gira sobre sí mismo (W proporcional al error con ganancia 2, entre 0'4 y 2) hasta que el error baja de 0'15 radianes. He usado dos umbrales distintos para evitar que llegue a alternar entre girar y avanzar. Durante este giro, el punto de mira es más lejano (12 metros) para que el coche se oriente según el recorrido y no según un tramo corto.  

### Llegada al destino y un nuevo destino  
Cuando el coche esté a menos de 3 celdas (3'75 metros) del destino, se detiene (V = 0 y W = 0) y espera. Al picar un nuevo punto de destino, se recalcula el campo.  

## Problemas durante la práctica  
A lo largo de la práctica me he ido encontrando distintos problemas.  
Un problema que tuve era que al cambiar el radio y el peso de penalización no cambiaba la ruta y el coche cortaba por las esquinas, acercándose mucho a ellas o chocando con ellas. El fallo que encontré fue que interpreté que la penalización se suma una sola vez en cada celda y, con pendiente menor que 1, solo desvía un poco el camino más corto. Por lo que mi solución fue engordar los obstáculos y apartar de las paredes los puntos de la ruta local.  
Otro problema que tuve fue que, al hacer que el coche girara sobre sí mismo, este, al terminar el giro, salía en diagonal, por lo que, si la calle era estrecha, se acababa acercando mucho a la pared o se chocaba con ella, por lo que mi solución fue elegir el vecino de mayor bajada por unidad de longitud, mirar más lejos al girar y reiniciar el filtro de W.  
También, al aumentar la velocidad, el coche empezó a hacer eses en las rectas, por lo que mi solución fue bajar la ganancia por encima de V = 15 y mirar más lejos cuanto más rápido va.  

## Conclusiones  
El taxi llega a los destinos que el usuario marca, se detiene al llegar y espera al siguiente destino. Lo que más me costó fue conseguir margen de seguridad en las esquinas y en las calles estrechas, manteniendo el campo como suma de los dos subcampos, la penalización por sí sola desvía poco el camino, así que tuve que apoyarme en el engordado de los obstáculos, en el desplazamiento de los puntos de la ruta local y en limitar la velocidad según el espacio disponible.  

## Vídeo del resultado  
He subido la velocidad del vídeo a un x5 para que sea más corto y ocupe menos espacio. En el vídeo se muestra como el coche toma una ruta hacia un destino para luego recalcular la ruta a otro destino seleccionado sin haber llegado al primero. También el pasar por calles estrechas y un giro sobre sí mismo.  



https://github.com/user-attachments/assets/7f5df0f1-663a-4928-88e1-6d6f4e2161b0





















