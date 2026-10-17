<h1 align="center">PRÁCTICA 2: FOLLOW LINE</h1>

<p align="center">
<img width="480" height="270" alt="image" src="https://github.com/user-attachments/assets/a83688dc-acc6-4317-8891-04aedff599ba" />
</p>
  
## 🔨 Objetivo de la práctica
En esta práctica configuré el funcionamiento de un coche de carreras para cualquier circuito, para ello dispuse de dos herramientas: un sensor y un controlador.
#### 📷​ Cámara:
  Por medio de la librería openCv, se realizó un filtrado HSV que nos mostraba la detección de la línea roja del circuito. Por ello, este sensor establece las dos primeras condiciones de funcionamiento del coche:
  
  -Detección de línea → si se detecta un número definido de píxeles la linea está siendo detectada por tanto circulamos
  -Búsqueda de la línea → el coche no detecta la linea roja o detecta un número de pixeles muy bajo, en ese caso reducimos el avance e iniciamos la búsqueda
    
#### 🎛️​ Controlador PID:
  Para poder estabilizar el seguido de línea del coche de carreras y su desempeño en los circuitos, se diseñó un controlador encargado de gestionar el giro del vehículo en función de lo detectado por el sensor.
  Asimismo, para mantener un mayor control sobre el vehículo y asegurar su movimiento en ámbitos de giro, el valor de la velocidad depende del valor de giro , pudiendo así reducir la velocidad en zonas de curva para no salirse del carril.
  Durante el inicio de la práctica , por medio del método de prueba y error, determiné los valores correspondientes para las ganancias del controlador. Además, también fui ajustando los valores de velocidad y giro máximo según su comportamiento en el circuito.
  
## ⭐ Arquitectura de la práctica
<img width="1272" height="105" alt="Screenshot from 2026-10-17 16-58-18" src="https://github.com/user-attachments/assets/13fd40ad-203f-43fc-8f14-8d52f129e23c" />

En este pequeño diagrama se puede apreciar de forma clara el funcionamiento del sistema y las relaciones entre los diferentes pasos/componentes. No obstante, dentro de la arquitectura de la práctica este diagrama inicial se divide en dos vertientes ya mencionadas, la detección de línea y la falta de esta, ambos estados muestran comportamientos diferentes, pero conexos.
#### 🟢 Detección de línea:
  Se considera detección de línea cuando la zona leída y filtrada ha detectado más de un umbral determinado de pixeles. En este caso se calcula en valor de W en función al error detectado, el valor de giro a su vez determina el de la velocidad, ya que ante giros muy amplios la velocidad deberá disminuir, mientras que en situaciones de recta o giros pequeños el coche podrá alcanzar su valor máximo de velocidad.
  Para regular el comportamiento del coche ante curvas muy repentinas como la del circuito de Nurburgring, he añadido una condición vinculada al valor de error que reduce la velocidad cuando se vislumbra una porción pequeña de la línea.
  
#### 🔴 Perdida de línea:
  Dentro de este caso hallamos dos posibilidades:
  
  - La línea no ha sido detectada antes: el coche gira con velocidad nula hasta detectar la línea del circuito.
  - Se ha perdido la línea: en este caso recurriremos al valor de giro detectado antes y haremos uso de su sentido para girar a menor velocidad en esa dirección y hallar nuevamente la línea. En el caso de que el valor de giro anterior haya sido muy amplio giramos en sentido contrario para volver a la línea, asumimos que nos hemos salido de la línea en dirección contraria a la que se dirigía.

## ⭐ Vídeos

A continuación se muestra el desempeño del coche en los circuitos básicos: SimpleCircuit, MonterealCircuit, MotmeloCircuit y NurburgringCircuit.

[Screencast from 2026-10-17 12-53-33.webm](https://github.com/user-attachments/assets/17a6b858-cc37-4a81-8995-8221c30fc86e)


https://github.com/user-attachments/assets/25a1c9cb-28ee-4410-a8e3-93018d6e941b



https://github.com/user-attachments/assets/e9a007d0-f9f2-4bd2-9d96-f4089dc62e14




https://github.com/user-attachments/assets/f490edbb-424b-470f-99e7-c29ac5ada6f9


## ⭐ Problemas detectados

El principal problema fue hallar una solución para las curvas repentinas de algunos de los circuitos, ya que causaban una circulación muy brusca y oscilante o en otros casos un desvio total de la línea, que gracias al método de recuperación implantado no tenía porque causar muchos problemas, pero que en el caso del segundo circuito podía resultar en un estancamiento o movimiento equivocado por parte del coche que detectaba las líneas rojas discontinuas que había a veces a los laterales de la carretera del circuito.

------------------------------------------------------------------------------------------------------------------------
## ⭐ REFERENCIAS
   Robotics Academy: https://jderobot.github.io/RoboticsAcademy/exercises/AutonomousCars/follow_line/
   
   Lectura con HSV: https://www.delftstack.com/es/howto/python/opencv-hsv/
   
