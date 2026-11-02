# Navegación local. Esquivar obstáculos  
En esta práctica tenemos que programar un coche de Fórmula 1 el cual debe esquivar distintos obstáculos distribuidos por el circuito gracias a la navegación local. Los objetivos de esta práctica es conseguir que el coche recorra el circuito en el menor tiempo posible mientras esquiva de forma fluida y segura los obstáculos del circuito. Para ello, se debe implementar el algoritmo VFF (Virtual Force Field), que gobierna la velocidad lineal (V) y la angular (W) en cada iteración.  
A la hora de comenzar, solo tenemos que centrarnos en la navegación local, ya que la navegación global ya viene resuelta, por lo que nos centramos en cómo llegar al siguiente subobjetivo sin chocar con lo que aparezca por el circuito. Un subobjetivo se da por alcanzado una vez que el coche esté lo bastante cerca, y entonces se pasa al siguiente. Todo esto lo tenemos en un bucle infinito, donde en cada iteración se procesa el láser, se calculan las fuerzas, se combinan y se traducen a V y W.  

## Las fuerzas  
El algoritmo VFF trata el coche como una partícula sobre la que actúan dos fuerzas, la fuerza atractiva y la fuerza repulsiva.  
· Fuerza atractiva: esta fuerza tira del coche hacia el subobjetivo actual.  
· Fuerza repulsiva: esta fuerza hace que cada obstáculo que ha sido detectado por el láser lo empuja en sentido contrario.  
La dirección que debe seguir el coche sale al combinar ambas fuerzas.  

## Proceso durante la elaboración de la práctica  
Lo primero que hay que tener en cuenta es el procesar el láser, el cual da 180 lecturas, una por cada grado, con la 90 apuntando de frente. A cada índice le restamos 90 para tener un ángulo en el que 0 es el frente y los positivos estén a la izquierda. Cuando hace una lectura con una distancia "infinita", la trato como si no hubiera un obstáculo cerca. También calculo la distancia mínima en la zona frontal, unos ± 15° (rayos 75 a 105), la cual luego uso para frenar.  
  
### Fuerza atractiva  





