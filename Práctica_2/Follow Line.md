# Follow Line  

## Introducción  
La práctica nos pide programar un coche de Fórmula 1 con una cámara frontal para que recorra un circuito marcado con una línea roja en el centro de la pista. El coche debe completar varios circuitos en el menor tiempo posible, manteniéndose en la línea intentando tener la menor oscilación posible. Además de que debe ser robusto, si en algún momento llega a perder la línea, debe saber encontrarla de nuevo.
Para poder realizar esta práctica, tenemos que tener en cuenta dos actuadores, la velocidad lineal (V) y la velocidad angular (W), los cuales cada uno cuenta con un PID, y una aplicación en forma de bucle infinito, donde en cada iteración se procesa la imagen y se deciden los valores de V y W.


## Desarrollo del código  
A la hora de resolver esta práctica, dentro de cada iteración del bucle se tenían que dar tres situaciones. La primera es buscar la línea, donde se filtra el rojo y se obtiene un punto. Luego se calcula el error, el cuánto se ha desviado el coche de la línea, siendo esto la diferencia entre ese punto y el centro de la imagen. Y por último, los dos PID de las dos distintas velocidades, la lineal (V) y la angular (W), las cuales convierten el error en W y V para saber cuánto gira y a qué velocidad va el coche.  

  Mi primer código fue uno simple y básico que hacía lo principal, solo tenía la P para el giro y la velocidad fija. Una vez conseguido el funcionamiento correcto del código, fui perfeccionándolo y añadiendo más cosas, cambiando siempre un parámetro cada vez para saber qué efecto tenía cada cambio en el coche.  

## De la imagen al error  
**Filtro de color en HSV:** El color lo teníamos que filtrar por HSV en vez de RGB, debido a que el tono y la saturación identifican el color mejor y de una forma más estable. El rojo tiene dos máscaras, una para el rango bajo y otra para el alto, las cuales se unen con un OR.  
  
**Banda de filas:** A la hora de usar la imagen, descarto el cielo y lo muy lejano, por encima del 40%, y también lo que está pegado al coche, por debajo del 84%. Una de las cosas que me ayudó a controlar mejor la oscilación fue calcular el centroide en una banda por delante del coche que empiece a corregir antes de llegar a desviarse, anticipando sus movimientos.  
  
**Centroide con momentos:** Usando `cv2.moments` se obtiene el centro de masa de los píxeles rojos, `cx`. El error es: `error = ancho/2 - cx`
  
## Control del giro (W) usando un PID  
PID significa:  
**P(proporcional):** El error de ahora. 
**I(integral):** El sumatorio del error, con signo.  
**D(derivativa):** La diferencia que hay entre el error actual y el de la iteración anterior.  
  
`W = KP * error + KI + Σerror + KD * (error - error_anterior)`   
  
La integral es la parte delicada, ya que si se deja acumular sin un control, arrastrará su efecto a las siguientes curvas. Por eso le puse un tope (INT_LIMIT) y, se pone a cero cuando el error baja de los 10 píxeles (RESET_ERROR), en cuanto el coche está prácticamente centrado se borra todo lo acumulado.  

## Control de la velocidad lineal (V) usando un PID  
En cuanto a la velocidad, no se mide su velocidad real sino que hay que decidir qué es el "error", por lo que lo definí como lo lejos que está el coche de una recta perfecta:  
  
 `error_velocidad = máx(|error cercano|, |error lejano|)`  
   
El error cercano es el mismo que usé para el giro, y el lejano sale de una distancia más alejada de la imagen. Esta ecuación vale cero cuando es una recta perfecta y crece al entrar en una curva o al desviarse.  
La salida de este PID no es la velocidad, sino cuánto debe frenar: 
  
`V = V_MAX - (KPV * e * KIV * Σe + KDV * Δe)`, limitada entre V_MIN y V_MAX.  
  
**P:** Cuanto más crece el error, más frena.  
**I:** Si se lleva un rato desviando el coche (curva larga), va frenando un poco más.  
**D:** Si el error crece rápido (el coche está entrando en una curva), frena antes. 

La velocidad frena de golpe, pero va acelerando poco a poco (ACCEL). La razón por la que frena rápido es porque es más seguro, al igual que acelerar poco a poco evita saltos bruscos de velocidad que puedan llegar a desestabilizar el giro.  

## Pérdida de línea  
Si no hay píxeles rojos, el coche gira hacia el lado que vio por última vez (last_direction), con una velocidad más baja, para ir controlado, hasta volver a encontrar la línea.  
  
El lado solo se llega a actualizar si el error es claro, para que el ruido no lo cambie. Una vez recuperada la línea, se reinician los acumulados y el error anterior, ya que si no, la derivada daría un tirón grande usando un valor antiguo.  

## Dificultades durante la práctica y su resolución  
A lo largo de la práctica puedo destacar con diferencia que lo más difícil de conseguir es que el coche realice bien las curvas.  
  
Al principio de la práctica al realizar un código más sencillo y con velocidad baja fija, no había problema a la hora de tomar las curvas, pero, en cuanto empecé a subir la velocidad y a añadir nuevas cosas en el código, el tomar las curvas se convirtió en un desafío. Al establecer una velocidad de 4 en el coche, este se salía al entrar en las curvas, así que hice que la velocidad dependiera del error para frenar en curva y, como el error llega tarde, lo que hice fue añadir una franja más lejana para poder anticiparme.  
  
Dentro de las curvas del circuito simple, la curva abierta, la más grande, fue la más complicada, ya que la línea central de la pista no dejaba de moverse entre las ruedas, es decir, no paraba de oscilar el coche hasta que descubrí que la velocidad variaba con el error y que la integral acumulada empujaba de más. Para poder resolverlo tuve que frenar el coche de forma más decidida, usando una integral muy pequeña que se llega a poner a cero con errores bajos y calculando el centroide en una banda por delante del coche. Al final, tuve que buscar un compromiso entre estabilidad y tiempo al elegir la velocidad mínima y máxima.  

## Conclusiones  
Al final de la práctica, he podido concluir que, en las curvas, lo que más ayudaba a resolver el problema era mirar más lejos y frenar a tiempo, más que tener que ir retocando los valores de P, I o D. Ir avanzando paso a paso, cambiando un parámetro cada vez y midiendo, me ayudó a encontrar el origen de cada problema.  

## Vídeos del código final
**Circuito simple**  
Para que el vídeo no se haga muy largo, le he subido la velocidad a x3, pero en el vídeo se puede ver tanto el tiempo real como el tiempo de simulación que tarda el coche en dar una vuelta.  

Tiempo simulación: 1 minuto y 10 segundos.  
Tiempo real: 3 minutos y 30 segundos.  
  




https://github.com/user-attachments/assets/8a5f3b7d-7a72-4a34-a928-9bfdd90593d7



  
**Circuito Montreal** 
Para que el vídeo no se haga muy largo, le he subido la velocidad a x6, pero en el vídeo se puede ver tanto el tiempo real como el tiempo de simulación que tarda el coche en dar una vuelta.  
  
Tiempo simulación: 3 minutos con 1 segundo.  
Tiempo real: 11 minutos con 7 segundos.  





https://github.com/user-attachments/assets/11a71eab-861b-4ce2-a014-471d6a76cd86



  
**Circuito Nurtburing**  
Para que el vídeo no se haga muy largo, le he subido la velocidad a x6, pero en el vídeo se puede ver tanto el tiempo real como el tiempo de simulación que tarda el coche en dar una vuelta.  

Tiempo simulación: 1 minuto y 17 segundos.  
Tiempo real: 7 minutos y 17 segundos.
  



https://github.com/user-attachments/assets/18cb71f7-e3d0-492f-b4f2-d10a906adc97




  
**Buscar línea**  


https://github.com/user-attachments/assets/21525585-f8ac-456b-bfcf-b338e0989218  
  







 
