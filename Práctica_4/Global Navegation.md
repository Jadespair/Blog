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
[IMAGEN MAPA DE PENALIZACION]  

### Subcampo de propagación  
Es una onda que sale del destino, con valor 0, y se extiende por las calles libres con el algoritmo de **Dijkstra**, el cual es una cola de prioridad que expande siempre la celda con menor coste. Se consideran los 8 vecinos, con coste 1 en horizontal y vertical y 1'4 (√2) en diagonal. Si una celda se alcanza por un camino más barato, se actualiza. Las celdas prohibidas no se llegan a expandir.  
La onda se propaga por todo el mapa alcanzable (no se detiene al llegar al coche), de modo que hay valor del campo en cualquier punto por el que pueda pasar el coche.  

### Campo total

