<h1 align="center">PRÁCTICA 1: BASIC VACUUM CLEANER</h1>

<p align="center">
<img width="320" height="270" alt="giphy" src="https://github.com/user-attachments/assets/37a6744d-75ca-4f7f-bee9-ba5c5c80716f" />
</p>
  
## 🔨 Objetivo de la práctica

En esta práctica se implementa un algoritmo de navegación para una aspiradora autónoma que recorre la mayor porción posible del mapa.

Primero se definió el patrón de movimiento, a partir de los movimientos básicos del robot:

- Avanzar
- Girar
- Retroceder

Tras probar un patrón simple, se buscaron ajustes para mejorar su eficacia. La práctica sugiere dos patrones de cobertura: **espiral** y **S**. En esta solución se utilizó la espiral, simulando así el movimiento de las aspiradoras de 2002. Además, este patrón parecía el idóneo para cubrir los espacios amplios de las salas.

A continuación se describe la evolución del algoritmo hasta la versión final.
  
## ⭐ Arquitectura de la práctica
<p align="center">
<img width="1143" height="595" alt="Screenshot from 2026-10-06 17-09-14" src="https://github.com/user-attachments/assets/a6bf37b0-d93f-4c41-a89a-0b1403778fbc" />
</p>

En el diagrama mostrado podemos ver de forma clara los estados del robot, y sus relaciones entre sí. Estos cuatro estados podemos dividirlos en dos tipos:

### Estados de avance:

  #### 🌀 ESTADO ESPIRAL:
  Se considera el movimiento principal del sistema, ya que es el que más abarca a limpiar en una sola ejecución, y por tanto el más eficiente para la práctica. Solamente se saldrá de este estado cuando se haya detectado un obstáculo a una distancia determinada (umbral_obstaculo), la cual se ha ido ajustando en función de los resultados vislumbrados en la simulación; o cuando la velocidad establecida por el calculo supere un umbral determinado (V_max).
  
  #### 🏃🏻‍♀️ ESTADO AVANZAR: 
  A lo largo de la práctica se ha ido cambiando su prioridad en la ejecución, pasando de ser el eje central del sistema en la primera implementación (cuyos resultados eran sorpresivamente favorables), a implementarse como una herramienta para favorecer la realización de espirales en espacios más amplios. Como se puede ver en el diagrama, se accede a este estado una vez se ha completado la espiral, en cuyo caso realizaremos un desplazamiento amplio ; y cuando durante el estado de giro se detecte un espacio libre en el área frontal, en ese caso el desplazamiento será menor. Ambos desplazamientos son dictados por un periodo de duración tempral establecido en cada caso.
                     
### Estados de maniobra:

Estos estados sirven para sacar al robot de situaciones de bloqueo causadas por obstáculos (esquinas, muebles...).

  #### ⏪ ESTADO RETROCEDER:
  Solamente se ejecuta en situaciones de bloqueo prolongado, cuando el robot ha permanecido girando durante un tiempo y no ha sido capaz de detectar espacios libres hacia los cuales avanzar por medio de el estado avanzar o el estado espiral. Además, esta acción solo se realizará durante cortos periodos de tiempo como medio para volver a estados de avance que continúen con la limpieza normal de la casa.

  ####  ↩️ ESTADO GIRAR:
  Es la maniobra más útil, la cual permite al robot cambiar de dirección y vislumbrar nuevos campos libres cuando se ha quedado bloqueado en una zona probablemente ya completada. Como medida de eficiencia se ha establecido un contador temporal para que dada una duración x, el robot cese el giro y explore otras salidas a través del estado de retroceso o el de avance en caso de que durante el giro se haya localizado un espacio libre. Además, si se observan distancias mayores al umbral de distancia al obstáculo se retorna al estado principal, el de espiral.
  Para la implementación de este estado veloré tres modos de ejecución:
  
  - Sentido de giro fijo
  - Sentido de giro aleatorio en cada ejecución
  - Sentido de giro en función del espacio libre
    
En un inicio implementamos el primer caso, cuyos resultados no fueron malos, pero sí mejorables. Seguidamente probamos la selección de sentido de giro aleatorio, la cual ofrecía resultados bastante buenos. A continuación probamos el tercer modo, que pese a ser el que parecía más óptimo no arrojó unos resultados mucho mejores que el anterior, incluso en ocasiones peores. Como consecuencia de ello, realicé una mezcla de estos dos últimos modos, aplicando el calculo de sentido de giro solo cuando entrabamos del estado de retroceso al de giro, intentando así optimizar el programa y mejorar las respuestas del robot cuando se quedaba encerrado. No obstante, este último método pese a dar resultados favorables y mejores que en el primer caso, a veces era superado en porcentaje de limpieza por la opción de sentido de giro aleatorio.
  
## ⭐ Vídeos
  (se subiran proximamente debido a problemas de edición/recorte)
  En esta sección se mostrarán los vídeos con el progreso de la práctica y la comparación de estos en función a los métodos usados como es el caso del video con senttido de giro aleatorio y el de sentido de giro mixto, que son los que más controversia crearon durante la elaboración del proyecto.

  ##### AVANZAR-GIRO-ATRÁS
  ....
  ##### ESPIRAL
  ....
  ##### ESPIRAL CON GIRO CONTROLADO
  ....
  ##### ESPIRAL CON GIRO ALEATORIO
  ....
  
## ⭐ Problemas detectados
  El mayor reto durante la práctica fue la elección de los parámetros que regirían la ejecución de la práctica, principalmente la elección de una distancia límite al obstáculo y la elección de periodos de tiempo correctos que limitasen la ejecución de giro y de otros estados ya mencionados. Este primer parámetro fue variando durante el programa iniciado con un valor de 0.5, el cual causaba la generación de huecos sin limpiar cerca de las paredes, motivo por el cual traté de reducir su valor a distancias de 0.25, 0.30 y 0.35. Finalmente, en el último modelo apliqué el valor de 0.35, que reducía estos huecos y no causaba problemas graves a la hora de limpiar cerca de obstáculos conflictivos como los muebles del salón (principalmente la mesita). En cuanto, a los parámetros de duración usados para la temporización de estados, estos fueron estimados en función a lo observado en las simulaciones.

------------------------------------------------------------------------------------------------------------------------
## ⭐ REFERENCIAS
   Robotics Academy: https://jderobot.github.io/RoboticsAcademy/exercises/MobileRobots/vacuum_cleaner
   
