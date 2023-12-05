# GLOBAL NAVIGATION
Esta práctica tenía dos fases, por un lado el mapeado y por otro la navegación.

La primera parte del mapeado consistía en rellenar un mapa según el coste el cual significaba la distancia del target marcado.

Lo primero que he hecho para esta parte es crear las variables necesarias para realizar el mapeado. Las variables claves eran la posicion inicial del coche y la posición del target.

```python 
      car = HAL.getPose3d()
      position = [car.x, car.y]
      car_position = tuple(MAP.rowColumn(position))
      car_position_map = car_position
      new_target = GUI.getTargetPose()    
      new_target_map = tuple(MAP.rowColumn(new_target))
```
A partir de aquí lo que había que hacer era generar ese mapa, empezando con coste 0 en el target y rellenar hasta donde estuviese el coche (ya que es hasta donde nos interesa)
Todo esto lo he hecho con una priority_queue la cual guardaba los puntos y sus costes, una lista con los puntos ya guardados y un conjunto de casillas las cuales estaban cerca de obstáculos.
Había que mirar a los vecinos de cada casilla para continuar esta parte y guardar cada coste en el grid.


```python
  cost = 0
  position = target_map
  
  priority_queue = queue.PriorityQueue()
  priority_queue.put((cost, position))
  
  visited = set()
  visited.add(target_map)
  
  #guardar los obstaculos
  obstacles_neighbors = set(tuple(obst) for obst in obstacles_neighbors)

  #hasta que no sea la posicion del coche no para
  while position != car_start: 
    cost, position = priority_queue.get()

    # obstaculo
    if map_array[position[1], position[0]] == 0:
      grid[position[1], position[0]] = 255
      continue


    # guardar vecinos
    neighbors = []
    
    for i in range(0,1):
        neighbors.append((position[0] + i ,position[1] + 1))
        neighbors.append((position[0] + i ,position[1] - 1))
        neighbors.append((position[0] + 1 ,position[1] + i))
        neighbors.append((position[0] - 1 ,position[1] + i))
        

    for neighbor in neighbors:
      
      #mirar si ha sido visitiado
      if neighbor in visited:
        continue
        
      #fuera del mapa
      if (neighbor[0] < 0 or neighbor[0] >= map_array.shape[1] or neighbor[1] < 0 or neighbor[1] >= map_array.shape[0]):
        continue
      
      #mirar si esta cerca de un objeto
      if tuple([neighbor[1],neighbor[0]]) in obstacles_neighbors:
        obstacle_weight = WEIGHT
      else:
        obstacle_weight = 0
        
      # mirar a ver si esta en la diagonal o no
      if neighbor[0] == position[0] or neighbor[1] == position:
        grid[neighbor[1], neighbor[0]] = cost + 1
      else:
        grid[neighbor[1], neighbor[0]] = cost + math.sqrt(2)
          
      # guardar
      priority_queue.put((grid[neighbor[1], neighbor[0]], neighbor))
      #agregar peso
      grid[neighbor[1], neighbor[0]] += obstacle_weight
      visited.add(neighbor)
```

[Grabación de pantalla desde 05-12-23 23:03:45.webm](https://github.com/jlozanot2021/Robotica_Movil_23.24/assets/102520615/dea970ac-29fe-4831-89e2-2dbc5e8f9f05)


Ahora con el mapa, lo único que hay que hacer es buscar la casilla cercana con menor coste.
Pimero para esto he relllenado una lista con las casillas cercanas de los obstáculos. A esto he de reconocer queu me ayudó un compañero ya que loque yo tenía hasta ese momento solo funcionaba si el target era muy cercano, pero si era lejano, dejaba de ir el programa.


```python
      obstacles = []
      for i in range(map_array.shape[1]):
          for j in range(map_array.shape[0]):
              if map_array[j][i] == 0:
                  obstacles.append((j, i))

```

```python
    for obstacle in obstacle_list:
        x, y = obstacle
        for i in range(-n, n + 1):
            for j in range(-n, n + 1):
                x_, y_ = x + i, y + j
                
                # ver si esta fuera
                if 0 <= x_ < map_array[0] and 0 <= y_ < map_array[1]:
                    obstacles_neighbors_list.add((x_, y_))

```
Si esta cerca de un obstáculo le he puesto de peso +30 para que evite estas zonas a toda costa.
Ahora lo único que falta es moverse. He hecho que busque la casilla de monor coste que esté alejada unas 5 casillas.
Además, he creado una lista para que no repita la misma casilla dos veces.
Por último, para el desplazamiento he usado lo mismo de la práctica anterior.

```python
#hasta que no llegue a la posicion no para
      while car_position != new_target:
        car = HAL.getPose3d()
        position = [car.x, car.y]
        car_position = tuple(MAP.rowColumn(position))
        neighbors = []
        #mira 5 casillas por delante
        lejos = 5
        for i in range(0,lejos):
          neighbors.append((car_position[0] + i ,car_position[1] + lejos))
          neighbors.append((car_position[0] + i ,car_position[1] - lejos))
          neighbors.append((car_position[0] + lejos ,car_position[1] + i))
          neighbors.append((car_position[0] - lejos ,car_position[1] + i))
        
        # buscar el de menor coste
        cost = 255
        next_position = car_position
        for neighbor in neighbors:
          #mirar si ya ha pasado por ahi
          if grid[neighbor[1],neighbor[0]] < cost:
            next_position = neighbor
            cost = grid[neighbor[1],neighbor[0]]
        path.append(next_position)
        
        #desplazarse
        next_position = gridToWorld(next_position)
        
        robot_x = HAL.getPose3d().x
        robot_y = HAL.getPose3d().y
        robot_yaw = HAL.getPose3d().yaw
        
        target_rel_x , target_rel_y = absolute2relative(next_position[0],next_position[1],robot_x,robot_y,robot_yaw)
        HAL.setW(target_rel_y)
        HAL.setV(target_rel_x)

        #si esta cerca parar
        if(abs(new_target[0] - HAL.getPose3d().x) <= 1 and (abs(new_target[1] - HAL.getPose3d().y) <= 1)):
          GUI.showPath(path)
          HAL.setW(0)
          HAL.setV(0)
          break
      
      finish = True

```

[Grabación de pantalla desde 05-12-23 23:00:48.webm](https://github.com/jlozanot2021/Robotica_Movil_23.24/assets/102520615/e6062256-0c37-4716-99b4-b439de97612a)

### Problemas
El mayor problema que he tenido en esta práctica ha sido el tema de los obstaculos y las casillas cercanas, ya que hasta que no me ayudaron con esto, mi coche iba pegado a la pared en todo momento.
Otro problema que tuve fue entender el la diferencia entre el mapa y gazebo ya que me costó el diferenciar ambos, lo cual era un problema para el tema de coordenadas

