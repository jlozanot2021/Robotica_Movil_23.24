# Obstacle Avoidance
### Objetivo de la práctica
La práctica consistía en pasar por unos puntos determinados y a su vez, esquivar los diferentes obstáculos qu ese ponían durante el camino.
Era necesario el uso del laser para así, implementar un sistema de atracción-repulsión.

### Implementación de VFF
El sistema de fuerzas virtuales tenía tres partes:
  - Fuerzas de atracción: Estas fuerzas eran proporcionadas por los diferentes targets que se encontraban alrededor del mapa.
    ```python

    currentTarget = GUI.map.getNextTarget() #cordenadas aboslutas hay que usar la funcion cuuret y pose (sale del hand)
    
    target_absolute_pose = [currentTarget.getPose().x, currentTarget.getPose().y]
    
    robot_x = HAL.getPose3d().x
    robot_y = HAL.getPose3d().y
    robot_yaw = HAL.getPose3d().yaw
    
    target_rel_x , target_rel_y = absolute2relative(target_absolute_pose[0],target_absolute_pose[1],robot_x,robot_y,robot_yaw)
    relative_target_show = target_rel_x,target_rel_y
    ```
  - Fuerzas de repulsión: Estas fuerzas resultaban de lo que llegase a detectar el laser.
    ```python
    def parse_laser_data (laser_data):
    laser = []
    for i, dist in enumerate(laser_data.values):
        if dist > 10:
            dist = 10
        angle = math.radians(i-90) # because the front of the robot is -90 degrees
        laser += [(dist, angle)]
    return laser
    ```
     Como estas fuerzas repelen, habia que hace una combersión para que restasen a la fuerza total y había que hacer también que las mas cercanas sean las mas grandes.
   
     ```python
      def obstacle_position ():
      mayor_distance = 0
      angulo = 0
      vector = []
      laser = parse_laser_data (HAL.getLaserData())
      for dist, angle in laser:
        x = math.cos (angle) * 1/dist * -1
        y = math.sin (angle) * 1/dist * -1
        v = (x , y)
        vector += [v]
      laser_mean = np.mean(vector, axis=0)
      return laser_mean
      ```
  - Fuerza resultante: Esta fuerza resulta de la unión de las dos anteriores.
     ```python
    avgForce = [(carForce[0] + obsForce[0]),(carForce[1] + obsForce[1])]
     ```
Como había que dar peso a los valores antes mencionados, ya que por mucha fuerza que atraiga el target, no debe chocarse con cualquier obstáculo qu ese le presente por el camino, estuve probando diferentes valores hasta llegar a la conclusión de que estos eran los mejores:
   ```python
    avgForce = [(1.35*carForce[0] + 2.5*obsForce[0])*1.35,(0.05*carForce[1] + 5*obsForce[1])]
   ```
Estos valores hacen que no se llegue a chocar y siga los objetivos marcados.
También he añadido que si el target se encuentra a mas de 2 metros siempre vaya a la misma velocidad y si se acerca empiece a frenar, todo esto para no pasarse del target
  ```python
  if(target_rel_x >= 2):
    target_rel_x = 2
  ```
Además de todo esto, he añadido que si llega a un mínimo local, pueda salir de el me diante esta implementación

    if (abs(1.35*carForce[0] + 2.5*obsForce[0]) < 0.01): 
      mlocal = 20
    elif (mlocal < 0):
      print("minimo local")
      mlocal -= 1
      HAL.setW(1)
      HAL.setV(3)
    else:
      HAL.setW(avgForce[1])
      HAL.setV(avgForce[0])

### Opciones descartadas
La verdad es que la única opción que probé y descarté fue un diagrama de estados en el que según los valores de repulsión y atracción tenía una velocidad angular y lineal diferente.
Apesar de que esta opción podría salir bien, me di cuenta que para que el coche hiciese bien la función, los casos deberían ser muchos y podrían hacer el código más difícil ( ya que seria muchos ifs ) y también, al probar me di cuenta que las variaciones de velocidad no le hacían muy bien al coche ya que una pequeña perturbación del laser podía hacer que se chocase o que las pareces le hiciesen ir más lento.


### Videos
A continuación, dos videos de como funciona el vehículo. El primero enseña que hace si el target se encuentra delante del coche y el segundo en una situación compleja.
[Screencast from 11-04-2023 06:48:37 PM.webm](https://github.com/jlozanot2021/Robotica_Movil_23.24/assets/102520615/085b450e-fc0e-4f02-9330-81592dfa82f5)

[Screencast from 11-04-2023 06:51:01 PM.webm](https://github.com/jlozanot2021/Robotica_Movil_23.24/assets/102520615/f962a73f-4152-4d8c-8c22-6e1583ad7aff)


### Observaciones

Los valores de if(target_rel_x <= 0.99 and target_rel_y <= 1.5): no estan puestos al azar ya que si paso de unos números se vuelve loco.

[Screencast from 11-04-2023 07:57:07 PM.webm](https://github.com/jlozanot2021/Robotica_Movil_23.24/assets/102520615/59c10447-24a5-41b5-abd5-846ccbb640f1)




