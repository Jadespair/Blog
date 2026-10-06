# Aspiradora básica autónoma
  En esta práctica tenemos que programar una aspiradora que tiene que recorrer una casa mientras la va limpiando, la mayor superficie posible, sin depender del mapa que se llegara a utilizar. El movimiento base de la aspiradora era una espiral, con la cual cubre bien las zonas abiertas, teniendo en cuenta los obstáculos que la aspiradora pudiese encontrarse, ya sean las paredes o los muebles de la casa, y sus giros se decidían de forma aleatoria cuando se detectaba un obstáculo. El enunciado nos pedía varios requisitos como, por ejemplo, que el autómata debe tener al menos tres estados, avanzar, retroceder y girar, también debe funcionar en un bucle infinito y no se debe usar la función *sleep* en ningún momento. 
    
  Mi primer objetivo para poder resolver la práctica es hacer una versión simple y sencilla del código requerido, el objetivo es que la aspiradora avance recto hasta que el láser detectará algo cerca y gire un rato. El resultado de este código fue bajo, ya que el robot se quedó atascado en una zona, sin poder salir de ella al haberse quedado atascado en un bucle por intento de salir del atasco. A partir de ahí fui mejorando el código poco a poco. Lo planteé con funciones *if/elif* haciendo una máquina de estados dentro del **while True**.  
    
  Cada vuelta del **while** empieza leyendo los 180 valores del láser y comprobando que están todos, porque si no, el programa da error al leer un rayo que no existe. Después dividí el cono de visión en tres distintos grupos, el centro (rayos del 60 al 120) y los dos laterales (del 140 al 179 y del 0 al 39), donde en cada uno me quedo con la distancia más corta, siendo este el obstáculo más cercano. También sume las lecturas de cada lado para saber en cuál hay más espacio libre. Si algo está a menos de 0'40 metros por delante o a menos de 0'22 metros, por un lado, la variable obstáculo se convierte en verdadera y entra la lógica de los estados.  
    
  En mi máquina de estados nos encontramos con los estados *ESPIRAL*, *AVANZAR*, *RETROCEDER* y *GIRAR*. En el estado *ESPIRAL* el robot avanza a una velocidad fija y el radio va en crecimiento según el tiempo que lleve en ese estado, con una velocidad angular **v / r**, de modo que el círculo se va abriendo. Una vez pasado 8 segundos el radio ya se ha hecho muy grande, así que pasa a *AVANZAR*, que simplemente va en línea recta. Estos dos estados comprueban si hay un obstáculo y, si lo llegará a haber, se cambia al estado *RETROCEDER*, donde da marcha atrás durante 0'2 segundos para separarse. En ese instante se decide hacia dónde girar, hacia el lado con más espacio, y cuánto tiempo, de forma aleatoria entre el intervalo de 1 a 2'5 segundos. Después, en el estado *GIRAR*, se gira ese tiempo y vuelve a la espiral si hay mucho espacio alrededor o vuelve al estado *AVANZAR*. Al no poder usar la función *sleep* para que el robot esté siempre activo, guardo la hora a la que empieza cada estado con time.time() y en cada vuelta compruebo cuánto lleva en él, así el bucle nunca se para. 
    
  Durante esta práctica, donde más dificultad encontré fue con el láser. Al principio el robot retrocedía sin haberse chocado con nada, y no entendía muy bien el por qué hasta que imprimí lo que leía el robot y vi que el láser devolvía *-inf* aunque no hubiera ninguna pared cerca, mi código lo tomaba como si hubiera tenido un choque. Estuve probando varias opciones para arreglarlo hasta que me di cuenta de que lo que tenía que hacer era ignorar esas lecturas inválidas y contarlas como distancia lejana. Otro problema que tuve fue que el robot dejaba una franja sin limpiar pegada a la pared, por lo que fui bajando la distancia a la que se detenía la aspiradora, pero, el hueco seguía ahí, por lo que opté por hacer un programa pequeño que solo avanzaba despacio y mostraba la lectura al chocar. Al hacerlo me daba el valor de 0'028 metros, porque el láser está montado en la parte delantera del robot y no en el centro como yo llegué a pensar. También intenté detectar los atascos comparando la posición del robot cada poco segundos, pero lo acabé descartando porque cuando el robot giraba en el sitio el detector saltaba sin estar atascado. Por último, al principio contaba vueltas del bucle para medir el tiempo, pero, comprobé que el bucle iba a más de 90 vueltas por segundo, por lo que lo pasé todo a segundos reales. 
    
  La diferencia de mi código final con el primer código de ensayo es que me enfoqué mejor en leer bien el sensor, vigilar las tres zonas en vez de solo el frente del robot, medir el tiempo sin pausas y girar siempre hacia donde haya más espacio.  

  A continuación enseño dos vídeos demostrativos del funcionamiento del código, uno nada más comenzar la aspiradora a funcionar y otro cuando ya lleva un rato limpiando:    
  He subido la velocidad del vídeo a x2 para que no sea tan lento de visualizar.  
    


https://github.com/user-attachments/assets/28949a31-edcd-4961-b017-55313f935d67  

  

https://github.com/user-attachments/assets/c6be6a98-626e-46ec-b279-716f0b79d777






    
