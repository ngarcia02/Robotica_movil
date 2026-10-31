<h1 align="center">PRÁCTICA 3: OBSTACLE AVOIDANCE</h1>


<p align="center">
<img width="356" height="200" alt="200" src="https://github.com/user-attachments/assets/fc601158-ebea-454e-8ff7-92e32ef78aed" />
</p>


## 🔨 Objetivo de la práctica
En esta práctica implementaremos un algoritmo VFF con el cual nuestro vehículo sea capaz de esquivar los obstáculos esparcidos por el circuito, a la máxima velocidad que pueda.
 
* Algoritmo VFF: cada obstáculo genera una fuerza de repulsión y el destino una fuerza de atracción, y la dirección del coche es la suma de todas las fuerzas.
* Además deberemos recorrer el circuito completo pasando por todos los waypoints sin colisionar.
* Objetivos opcionales: ir lo más rápido posible, elegir la ruta más segura, tratar obstáculos en movimiento y ser robusto ante situaciones de indecisión (suma de vectores nula).

En mi solución se ha priorizado ir rápido y completar la vuelta sin colisionar, con una velocidad que se adapta a la distancia al obstáculo frontal.

## 📏Fundamentos teóricos
Para la realización de la simulación hemos tenido en cuenta los siguientes fundamentos.

### 🌍 Navegación global y local

La navegación de robots abarca todas las tareas necesarias para llevar un robot de un punto A a un punto B de forma autónoma y sin colisiones: localización, evitación de obstáculos, mapeo, planificación y exploración. El problema de planificación de rutas se aborda de dos formas:

- Navegación global: usa un mapa del entorno para planificar la ruta completa, con algoritmos como Dijkstra, Best First o A*.
- Navegación local: la ruta global se divide en waypoints y el robot los recorre ajustando su trayectoria de forma dinámica según lo que percibe.

Esta práctica se centra en la navegación local con VFF.

### Virtual Force Field

El algoritmo se basa en tres pasos:

  1. El robot asigna un vector de atracción que apunta hacia el waypoint.
  
  2. El robot asigna un vector de repulsión a los obstáculos, calculado a partir de las lecturas de los sensores y que apunta en sentido contrario a ellos. Se obtiene sumando los vectores correspondientes a cada lectura.
  
  3. El robot sigue el vector resultante de sumar el vector objetivo y el vector de obstáculos.

### Inconvenientes del VFF

- El robot tiende a oscilar en pasillos estrechos, cuando recibe a la vez vectores de repulsión de lados opuestos.
  
- Puede no ser capaz de entrar en pasillos estrechos.
  
- Cuando la suma de fuerzas se anula, la dirección resultante es indeterminada (situación de indecisión).

## 🛠️ Entorno y herramientas

La práctica se desarrolla en Python sobre el entorno de simulación de Robotics Academy (Gazebo) con la interfaz web WebGUI. Se utiliza la implementación basada en HAL:
* `HAL.getPose3d()` para la posición (x, y) y la orientación (yaw) del coche.
* `HAL.getLaserData()` para las 180 lecturas del láser, con una resolución de un rayo por grado. El índice 0 es el rayo de la derecha, el 90 es el frontal y el 179 el de la izquierda.
* `HAL.setV()` y `HAL.setW()` para las velocidades lineal y angular.
* `WebGUI.getNextTarget()`, `getPose()` y `setReached(True)` para obtener el waypoint actual y marcarlo como visitado.
* `WebGUI.showForces()` y `WebGUI.showLocalTarget()` para depurar visualmente las fuerzas (verde: atracción, roja: obstáculos, negra: resultante) y el objetivo.
* `Frequency.tick()` para regular la frecuencia del bucle (50 Hz).

Se trabaja con dos sistemas de coordenadas: el absoluto, con origen en la línea de meta del circuito, y el relativo al coche, donde X positivo es hacia delante e Y positivo hacia la izquierda.

## ⭐ Arquitectura de la práctica

En cada iteración del bucle principal (50Hz) se hace lo siguiente:

  1. Se obtiene el waypoint actual y la pose del coche.
  2. Se lee el láser y se calcula el vector de repulsión y la distancia frontal.
  3. Se calcula el vector de atracción en coordenadas relativas al coche.
  4. Se combinan los vectores para obtener la dirección.
  5. Se calcula la velocidad angular y lineal.
  6. Se comprueba si se ha alcanzado el waypoint y se envían las órdenes a los motores.
     
### 🧲 Vector de atracción: 
Se resta la posición del coche a la del waypoint y se rota el resultado con el ángulo del coche para expresarlo en un sistema de referencia (función absolute2relative). Después se normaliza, de modo que el módulo de la atracción siempre es 1 (multiplicado por el peso W_ATRACTION).
 ```python 
  # Obtenemos el vector de atracción
   x_car, y_car = absolute2relative(target_x,target_y, position_x, position_y, angle)
  # normalizamos
   x_car_n = x_car/distance
   y_car_n = y_car/distance
 ```
           
La rotación es necesaria para que el vector apunte siempre hacia el waypoint y no hacia delante del coche.
No se ha incluido una comprobación explícita de distance > 0 antes de normalizar el vector de atracción. El waypoint se marca como alcanzado cuando la distancia baja de UMBRAL_VISITADO (1.5 m), y el coche avanza como máximo unos 0.14 m por iteración, por lo que la distancia al waypoint actual nunca se aproxima a cero en la práctica.

### 🔴 Vector de repulsión:
Cada uno de los 180 rayos del láser se convierte a coordenadas polares, con un ángulo 0 al frente y positivo a la izquierda. Cada rayo con una lectura válida y a menos de D_MAX genera un vector de repulsión que apunta en sentido contrario al obstáculo, con magnitud inversamente proporcional al cuadrado de la distancia. Aplicamos este  umbral para no tener en cuenta los rayos muy lejanos y centrarnos en los obstáculos más cercanos. Además, se descartan las lecturas infinitas o nulas. Posteriormente sumamos todos los vectores:
```python
    laser_data = HAL.getLaserData()
       if len(laser_data.values) > 0:
           laser_polar = parse_laser_data(laser_data)
           frontal = min(laser_data.values[65:116])
           #obtenemos la distancia y el ángulo
           for dist, ang in laser_polar:
               if (dist != math.inf and dist != 0 and dist < D_MAX):
                   obs_x += (-(math.cos(ang)))/(dist*dist)
                   obs_y += (-(math.sin(ang)))/(dist*dist)
```
He elegido 1/d² en lugar de 1/d para que los obstáculos cercanos pesen mucho más que los lejanos y el coche no se desvíe por paredes que aún están muy lejos.

### ⬆️ Vector de dirección:

Sumar directamente la atracción y la repulsión no funciona bien: casi siempre hay obstáculos en el semicírculo frontal del láser, por lo que la componente X de la repulsión apunta casi siempre hacia atrás y distorsiona la dirección resultante. Como el coche debe avanzar siempre, la componente X apenas influye en el movimiento, y es la componente Y la que controla el giro y, por tanto, la evitación de obstáculos.

Como consecuencia en mi código se han adoptado tres casos según la distancia frontal ( frontal, el mínimo de las lecturas entre los índices 65 y 115):

* Camino despejado: el obstáculo está situado a más de 9 metros de distancia. Esto se aplica principalmente en rectas largas y despejadas. Se usa la componente X de la atracción y la suma de las componentes Y de ambas fuerzas.
* Camino semi-despejado: el obstáculo está entre 5 y 9 metros de distancia del coche. Reducimos un poco la velocidad, para estar preparados según nos vayamos acercando.
* Camino obstaculizado: el obstáculo está a menos de 5 metros. Volvemos a reducir la velocidad para evitar la colisión.

En los dos últimos casos se suman ambas fuerzas completas, incluida la componente X de la repulsión. Esto hace que el giro sea más pronunciado cuando el obstáculo está cerca.

### 🔄 Control de giro:
La velocidad angular es proporcional al ángulo del vector dirección:
    
    w = K · atan2(avg_y, avg_x)

## 📍 Gestión de waypoints:
Cuando la distancia al waypoint baja de UMBRAL_VISITADOS (1.5m), se marca como alcanzado con setReached(True) y en la siguiente iteración getNextTarget() devuelve el siguiente.


## ⭐ Limitaciones:
* Oscilaciones: como en cualquier VFF, el coche puede oscilar si recibe repulsión simultánea de ambos lados. Esto se ha mitigado ajustando los pesos y la ganancia K.

* Indecisión: no se ha implementado un tratamiento específico para cuando la suma de fuerzas se anula. Como prueba, se intentó que el coche eligiera el lado con más espacio libre en esas situaciones, pero en mis pruebas el resultado fue peor que la versión final, por lo que se descartó.

* Dependencia de los waypoints: en las curvas, el waypoint puede quedar fuera del carril, y la atracción tira del coche hacia la pared. La repulsión compensa este efecto, pero es una de las causas de los temblores cuando los pesos no están bien equilibrados.

* Valores fijos: los umbrales y velocidades están ajustados para este circuito y podrían necesitar reajuste en otro

## ⭐ Vídeos

En este primer vídeo podemos ver la ejecución normal del coche en el circuito:

https://github.com/user-attachments/assets/1b7812b3-9f06-4d69-9145-04db877ed193

Para una comprobación de la robustez del programa he movido algunos de los obstáculos para ver la reacción del coche ante ellos:

https://github.com/user-attachments/assets/96e12b37-92e0-46b8-8181-55f63448cebf

------------------------------------------------------------------------------------------------------------------------
## ⭐ REFERENCIAS
- Robotics Academy (JdeRobot), ejercicio de navegación local con VFF: <https://jderobot.github.io/RoboticsAcademy/exercises/AutonomousCars/obstacle_avoidance>
- J. Borenstein y Y. Koren, "Real-time obstacle avoidance for fast mobile robots", *IEEE Transactions on Systems, Man, and Cybernetics*, 19(5), 1989.
- J. Borenstein y Y. Koren, "The vector field histogram: fast obstacle avoidance for mobile robots", *IEEE Transactions on Robotics and Automation*, 7(3), 1991.
   
