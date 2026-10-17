# Follow Line  

## Introducción  
La práctica nos pide programar un coche de Fórmula 1 con una cámara frontal para que recorra un circuito marcado con una línea roja en el centro de la pista. El coche debe completar varios circuitos en el menor tiempo posible, manteniéndose en la línea intentando tener la menor oscilación posible. Además de que debe ser robusto, si en algún momento llega a perder la línea, debe saber encontrarla de nuevo.
Para poder realizar esta práctica, tenemos que tener en cuenta dos actuadores, la velocidad lineal (V) y la velocidad angular (W), los cuales cada uno cuenta con un PID, y una aplicación en forma de bucle infinito, donde en cada iteración se procesa la imagen y se deciden los valores de V y W.


## Desarrollo del código  
A la hora de resolver esta práctica, dentro de cada iteración del bucle se tenían que dar tres situaciones. La primera es buscar la línea, donde se filtra el rojo y se obtiene un punto. Luego se calcula el error, el cuánto se ha desviado el coche de la línea, siendo esto la diferencia entre ese punto y el centro de la imagen. Y por último, los dos PID de las dos distintas velocidades, la lineal (V) y la angular (W), las cuales convierten el error en W y V para saber cuánto gira y a qué velocidad va el coche.  

  Mi primer código fue uno simple y básico que hacía lo principal, solo tenía la P para el giro y la velocidad fija. Una vez conseguido el funcionamiento correcto del código fui perfeccionándolo y añadiendo más cosas, cambiando siempre un parámetro cada vez para saber qué efecto tenía cada cambio en el coche.  

## De la imagen al error  
**Filtro de color en HSV:** El color lo teníamos que filtrar por HSV en vez de RGB, debido a que el tono y la saturación identifican el color mejor y de una forma más estable. El rojo tiene dos máscaras, una para el rango bajo y otra para el alto, las cuales se unen con un OR.  
  
**Banda de filas:** A la hora de usar la imagen, descarto el cielo y lo muy lejano, por encima del 40%, y también lo que está pegado al coche, por debajo del 84%. Una de las cosas que me ayudó a controlar mejor la oscilación fue calcular el centroide en una banda por delante del coche que empiece a corregir antes de llegar a desviarse, anticipando sus movimientos.  
**Centroide con momentos:** Usando _cv2.moments_ se obtiene el centro de masa de los píxeles rojos, _cx_. El error es:   
  _error = ancho/2 - cx_  

## Control del giro (W) usando un PID  
PID significa:  
**P(proporcional):** el error de ahora.  
**I(integral):** el sumatorio del error, con signo.  
**D(derivativa):** la diferencia que hay entre el error actual y el de la iteración anterior.  
#### W = KP * error + KI + Σerror + KD * (error - error_anterior)   
La integral es la parte delicada, ya que si se deja acumular sin un control arrastrará su efecto a las siguientes curvas. Por eso le puse un tope (INT_LIMIT) y, se pone a cero cuando el error baja de los 10 píxeles (RESET_ERROR), en cuanto el coche está prácticamente centrado se borra todo lo acumulado.  

## Control de la velocidad lineal (V) usando un PID  
En cuanto a la velocidad, no se mide su velocidad real sino que hay que decidir qué es el "error", por lo que lo definí como lo lejos que está el coche de una recta perfecta:  
#### error_velocidad = máx(|error cercano|, |error lejano|)  
El error cercano es el mismo que usé para el giro, y el lejano sale de una distancia más alejada de la imagen. Esta ecuación vale cero cuando es una recta perfecta y crece al entrar en una curva o al desviarse.  
La salida de este PID no es la velocidad, sino cuánto debe frenar: 
  
**V = V_MAX - (KPV * e * KIV * Σe + KDV * Δe)**, limitada entre V_MIN y V_MAX.  
  
**P:** cuanto más crece el error, más frena.  
**I:** si se lleva un rato desviando el coche (curva larga), va frenando un poco más.  
**D:** si el error crece rápido (el coche está entrando en una curva), frena antes. 

La velocidad frena de golpe pero va acelerando poco a poco (ACCEL). La razón por la que frena rápido es porque es más seguro, al igual que acelerar poco a poco evita saltos bruscos de velocidad que puedan llegar a desestabilizar el giro.  

## Pérdida de línea  
Si no hay píxeles rojos, el coche gira hacia el lado que vio por última vez (last_direction), con una velocidad más baja, para ir controlado, hasta volver a encontrar la línea.  
El lado solo se llega a actualizar si el error es claro, para que el ruido no lo cambie. Una vez recuperada la línea se reinician los acumulados y el error anterior, ya que si no, la derivada daría un tirón grande usando un valor antiguo.  

## Dificultades durante la práctica y su resolución  
A lo largo de la práctica puedo destacar con diferencia que lo más difícil de conseguir es que el coche realice bien las curvas.  
Al principio de la práctica al realizar un código más sencillo y con velocidad baja fija, no había problema a la hora de tomar las curvas pero, en cuanto empecé a subir la velocidad y a añadir nuevas cosas en el código, el tomar las curvas se convirtió en un desafío.  
