# Navegación local. Esquivar obstáculos  
En esta práctica tenemos que programar un coche de Fórmula 1 el cual debe esquivar distintos obstáculos distribuidos por el circuito gracias a la navegación local. Los objetivos de esta práctica son conseguir que el coche recorra el circuito en el menor tiempo posible mientras esquiva de forma fluida y segura los obstáculos del circuito. Para ello, se debe implementar el algoritmo **VFF** (Virtual Force Field), que gobierna la velocidad lineal (V) y la angular (W) en cada iteración.  
A la hora de comenzar, solo tenemos que centrarnos en la navegación local, ya que la navegación global ya viene resuelta, por lo que nos centramos en cómo llegar al siguiente subobjetivo sin chocar con lo que aparezca por el circuito. Un subobjetivo se da por alcanzado una vez que el coche esté lo bastante cerca, y entonces se pasa al siguiente. Todo esto lo tenemos en un bucle infinito, donde en cada iteración se procesa el láser, se calculan las fuerzas, se combinan y se traducen a V y W.  

## Las fuerzas  
El algoritmo VFF trata el coche como una partícula sobre la que actúan dos fuerzas, la fuerza atractiva y la fuerza repulsiva. 
  
+ **Fuerza atractiva:** esta fuerza tira del coche hacia el subobjetivo actual.  
+ **Fuerza repulsiva:** esta fuerza hace que cada obstáculo que ha sido detectado por el láser lo empuje en sentido contrario.  
  
La dirección que debe seguir el coche sale al combinar ambas fuerzas.  

## Proceso durante la elaboración de la práctica  
Lo primero que hay que tener en cuenta es el procesar el láser, el cual da 180 lecturas, una por cada grado, con el 90 apuntando de frente. A cada índice le restamos 90 para tener un ángulo en el que 0 es el frente y los positivos estén a la izquierda. Cuando hace una lectura con una distancia "infinita", la trato como si no hubiera un obstáculo cerca. También calculo la distancia mínima en la zona frontal, unos ± 15° (rayos 75 a 105), la cual luego uso para frenar.  
  
### Fuerza atractiva  
Como el subobjetivo viene dado en coordenadas absolutas del mapa, paso esas coordenadas a los ejes del coche: x hacia delante e y hacia la izquierda, le resto la posición del coche y rotamos la diferencia según su dirección (yaw).  
  
`atr_x = dif_x * math.cos(coche.yaw) + dif_y * math.sin(coche.yaw)`    
`atr_y = -dif_x * math.cos(coche.yaw) + dif_y * math.cos(coche.yaw)`  
  
Después normalizo el vector para que tenga módulo 1, para que la atracción solo aporte dirección y no dependa de lo lejos que esté el subobjetivo.  

### Fuerza repulsiva  
Al haber un obstáculo cerca, cada rayo genera un vector que apunta en sentido contrario, con módulo **1/d^2**. Con el cuadrado, un obstáculo cercano repele mucho más que uno lejano. Solo llego a tener en cuenta los rayos a menos de 5 metros, y limito la distancia mínima a 0.15 metros para no tener que dividir por casi cero. Sumo los 180 vectores, los escalo y limito la componente lateral para que no haya reacciones exageradas.  

### Combinar las fuerzas  
La repulsión suma casi siempre un vector que apunta hacia atrás, porque el láser solo mira al frente y todos sus rayos empujan hacia atrás. Si se suma tal cual a la atracción, el coche frena o duda sin ningún motivo. Por eso el avance lo marca solo la atracción, con un mínimo de 0.6, y la repulsión solo influye en el giro, en la componente lateral:  
  
`fuerza_x = max(atr_x, 0.6)`  
`fuerza_y = atr_y + rep_y`  
  
Si hay un obstáculo a menos de 1 metro delante y la fuerza lateral es casi nula, se fuerza un giro hacia el lado con más hueco.  

### Velocidades V y W  
El ángulo de la fuerza resultante respecto al frente del coche es el error de rumbo que hay que corregir.  
  
+ **V:** parto de la velocidad máxima para reducirla según dos factores. El primer factor es cuánto hay que girar y el segundo factor es lo cerca que está el obstáculo por delante. En la recta libre la velocidad va al máximo y frena al acercarse a un obstáculo o al entrar en una curva cerrada.  
+ **W:** controlador proporcional + derivativo sobre ese ángulo. Con el término derivativo se frena el giro cuando el ángulo ya está cambiando rápido. La velocidad se satura y se suaviza mezclándola con el giro de la iteración anterior.  
  
### Cambio de subobjetivo  
Cuando el coche esté a menos de 3 metros del subobjetivo, se marca como alcanzado sin detener el coche, así consigo que no frene en cada punto y el movimiento sea más fluido.  

## Problemas durante el proceso  
Al sumar la repulsión al avance, esa componente hacia atrás hundía la variable `fuerza_x` hasta el mínimo y el ángulo se volvía sensible, causando que el coche hiciera zigzag mientras se movía. Para conseguir resolver este problema tuve que dejar el avance en menos atracción y añadir el término derivativo y el suavizado del giro. Aprendí también que el factor de suavizado es el peso del giro anterior, a 1.0, el giro nunca cambia y el coche no gira.  
Este problema del zigzag volvió a salir una vez que empecé a subir la velocidad del coche para que pudiera completar el circuito en el menor tiempo posible, juntándolo con que se chocaba con los obstáculos, ya que reaccionaba con el mismo retardo, pero recorría más distancia. Por eso subí la velocidad poco a poco y añadí un frenado por distancia frontal y por giro, y un mínimo de velocidad para no perder tiempo de vuelta.  

## Conclusiones  
El algoritmo VFF depende mucho del ajuste de sus parámetros, y combinar bien las fuerzas importa más que ajustarlas. En esta práctica he visto que muchos de los que yo pensaba que eran errores del algoritmo, en realidad, el problema era cómo combinaba las fuerzas.  

## Vídeo del resultado  
He puesto la velocidad de reproducción en x3 para que el vídeo no se haga tan largo, pero se puede ver el tiempo de simulación y el tiempo real en el vídeo.  
Tiempo Simulación: 47'8 segundos    
Tiempo Real: 3 minutos y 9 segundos    




https://github.com/user-attachments/assets/f4e93e7c-a46a-4681-9c53-df5fc3c61414








