<h1 align="center">PRÁCTICA 4: GLOBAL NAVIGATION</h1> 

<p align="center">  
<img width="480" height="480" alt="giphy" src="https://github.com/user-attachments/assets/a08bdfb6-b339-4588-a43a-093be6789a28" />
</p>

## 🔨 Objetivo de la práctica

En esta práctica se programa un taxi autónomo, equipado con un sensor GPS y un mapa de la ciudad, capaz de planificar su navegación desde el punto en el que se encuentra hasta un destino que el usuario marca seleccionado en el mapa. Una vez alcanzado el destino, el taxi espera a que se le indique un nuevo objetivo.

Para la planificación se utiliza el algoritmo de Gradient Path Planning (GPP), un ejemplo de plan como recurso: se calcula un campo escalar sobre el mapa que el robot puede consultar en cualquier momento para saber hacia dónde moverse. Este campo es la suma de dos componentes:

- Subcampo de propagación a destino: coste de llegar al destino desde cada celda.
- Subcampo de penalización: coste extra por cercanía a obstáculos.

Además de planificar, la aplicación debe pilotar el coche siguiendo el gradiente del campo, gobernando de forma reactiva la velocidad lineal (V) y la angular (W) en cada iteración.

## ⭐ Arquitectura de la práctica

El programa se divide en dos bloques: la planificación, que se ejecuta una sola vez al inicio y cada vez que cambia el destino, y la navegación, que se ejecuta en cada iteración del bucle reactivo.

### Bloque de planificación:
##### 🗺️ MAPA Y REJILLA:

El mapa es una imagen binaria de 400x400 celdas (255 = libre, 0 = obstáculo). Se trabaja directamente sobre la rejilla y se usan worldToGrid y gridToWorld para convertir entre coordenadas del mundo y de la rejilla, teniendo en cuenta que la celda (i, j) de la imagen se accede como (j, i).

##### 🧱 SUBCAMPO DE PENALIZACIÓN:

Se calcula una sola vez, ya que solo depende del mapa y no del destino. Alrededor de cada obstáculo se aplica una penalización por anillos  hasta un radio de RADIO celdas: cuanto más cerca del obstáculo, mayor es el valor de penalización. Si una celda está cerca de varios obstáculos se conserva la mayor penalización, no la suma. Como optimización, se descartan los obstáculos rodeados únicamente por otros obstáculos (interior de las paredes), ya que no aportan nada.
En la asignación de valores de penalización se tuvo en cuenta su efecto sobre el coche a la hora de navegar, por ello se aumento su valor, ya que inicialmente al no tener penalizaciones muy altas el coche se pegaba mucho a las paredes, lo que causaba choques con esquinas frecuentemente.

##### 🌊 SUBCAMPO DE PROPAGACIÓN (FRENTE DE ONDA):

Partiendo del destino con coste 0, se expande una onda con una cola de prioridad (heapq), asignando a cada vecino el coste acumulado. Los movimientos diagonales cuestan más que los ortogonales (1.4 frente a 1) y a cada paso se le suma la penalización de la celda a la que se entra, de modo que el campo final integra ambos subcampos. Las paredes y las celdas inalcanzables quedan con coste infinito. El campo solo se recalcula cuando el usuario cambia de destino.

### Bloque de pilotaje (bucle reactivo):
##### 🎯 SELECCIÓN DEL PUNTO OBJETIVO LOCAL:

Desde la celda actual del coche se sigue el descenso por el gradiente (el vecino de menor coste) acumulando distancia recorrida hasta alcanzar una distancia de lookahead (L_DIST). La celda a la que se llega es el punto objetivo local. El lookahead crece con la velocidad, para no pasar de largo en las curvas ni quedar atrapado en un mínimo local.

Otro factor muy importante durante la navegación al destino es el establecimiento de la velocidad y el giro del vehículo:
###### 🚗 CONTROL DE V Y W:
- Velocidad lineal: parte de una velocidad máxima V y se reduce linealmente al acercarse al destino (D_FRENADO), y también en función del error angular, de modo que en las curvas el coche baja la velocidad. (Este frenado según se acerca a destino fue añadido ya que con el objetivo de hacer que el coche fuera más rápido en muchas ocasiones pasaba de largo del objetivo, y esta fue la opción más optima)
- Velocidad angular: se calcula a partir de la curvatura hacia el punto objetivo local (w = v · 2·sin(α) / dist). Se limita a W_MAX, para evitar choques con paredes cercanas sobre todo en los pasillos estrechos.
- Error angular grande: si el error supera ANGLE_PARAR, el coche se detiene y gira sobre sí mismo hasta alinearse.
###### 📍 GESTIÓN DE DESTINOS:
- Si el destino no es transitable, o no hay ruta hasta él, el coche se detiene y espera otro destino.
- Se considera alcanzado cuando la distancia al destino es menor que UMBRAL_DESTINO. Entonces el coche se detiene y espera un nuevo destino, para evitar que los valores de V e W calculados en la anterior ejecución causen problemas en la nueva navegación.

## Problemas detectados

Durante la realización de la practica fui consciente de la importancia de establecer correctamente los valores de penalización a las celdas cercanas a los obstáculos, ya que un valor inferior generaba choques con esquinas entre navegación y navegación o durante una misma navegación, y valores muy altos podían causar problemas para transitar por la mejor ruta.
Asimismo, un problema visual más que del ejercicio fue que al seleccionar el objetivo la x mostrada por pantalla se situaba al lado derecho de la selección, lo que al inicio me causo confusión.

## ⭐ Vídeos

En esta sección se mostrarán los vídeos con el progreso de la práctica:

En este primer vídeo mostraremos la ejecución normal del programa:

[Screencast from 2026-11-19 16-23-27.webm](https://github.com/user-attachments/assets/1339011b-c445-4c94-984a-17c7b4520128)


En este video mostraremos la respuesta del vehículo ante cambios de destino durante la navegación:

[Screencast from 2026-11-19 16-24-58.webm](https://github.com/user-attachments/assets/3e9ba68b-65b4-4066-b574-e4c328f135de)

En este último vídeo mostraremos como el robot no navega a puntos no transitables:


[Screencast from 2026-11-19 16-29-01.webm](https://github.com/user-attachments/assets/fcf511c7-91eb-4b76-b6bc-4c509f7c9781)




⭐ REFERENCIAS

Robotics Academy: https://jderobot.github.io/RoboticsAcademy/exercises/AutonomousCars/global_navigation/
