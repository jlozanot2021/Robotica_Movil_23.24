# Monte Carlo Laser Localization

El objetivo de esta práctica es obtener la localización del robot mediante la fabricación de diferentes partículas, las cuales, a partir de la comparación de su láser con el del robot real, su posición se solape con la del real.

En primer lugar, había que crear todas estas partículas de manera aleatoria en el mapa. Para esta función había que crear un determinado numero de partículas de las cuales había que comprobar si estaban en el mapa y no dentro de objetos. En el caso de que su posición no fuese valida, esta partícula sería descartada y no sumaría a la cantidad de partículas creadas.

```python
particle = np.random.uniform(low=[x_low, y_low, 0.0],
                                    high=[x_high, y_high, 2*np.pi],
                                    size=(3,))
  x , y , theta = particle
  x_w, y_w, t_w = MAP.worldToMap(x, y, theta)
  if (map_array[y_w][x_w] != 0):
      i += 1
      particles.append(particle)
```

Ahora, había que añadir unos pesos a cada partícula para comprobar cual es la que está más cerca del robot real. Aquí he reducido la cantidad del laser de ambos a un 10% para que todo se realice mucho mas veloz. Se restan las medidas de los láseres y de esta manera se puede sacar ese peso deseado para cada partícula.
```python
diference = abs(laser_data_mini - laser_data_particle)

mean_particle = np.mean(diference**2)

if mean_particle != 0:
    weight.append(1/mean_particle)
else:
    weight.append(0)
```
Había que estar atento a si el mean_particle era 0, lo cual daría un error (1/0 es un error).

Con las partículas ya generadas y sus pesos formados solo nos queda que se elijan a las que tienen los laseres mas parecidos al robot real. Se remplazan las partículas las cuales no tienen medidas similares por unas "cercanas" a las más parecidas al láser del robot real.  
```python
selected_idx = np.random.choice(N_PARTICLES, replace=True, size=N_PARTICLES, p=weights)
particles = old_particles[selected_idx]
```

Antes he hablado de posiciones cercanas entre las partículas ya que no se pueden crear en el mismo lugar debido a que no tendría ningún sentido. En este punto entra la funcion propagate_particles la cual hace que las partículas se muevan a la vez que el robot pero con un cierto grado de aleatoriedad para que no esten todas en el mismo lugar.
```python
    dx = dt * LINEAR_VEL * np.cos(yaw)
    dy = dt * LINEAR_VEL * np.sin(yaw)
    dyaw = dt * ANGULAR_VEL
    # Add this movement to the particle, with an extra Gaussian noise
    particle[0] += dx + np.random.normal(0.0, 0.02)
    particle[1] += dy + np.random.normal(0.0, 0.02)
    particle[2] += dyaw + np.random.normal(0.0, 0.01)
```
Gracias a esto, las partículas se mueven al compas del robot pero formando una gran nube de poartículas que ayudan a que se pueda localizar el robot.
Finalmente, se repite el seleccionar las partículas mas parecidas y eliminar las diferentes y la propagación hasta que el robot real se enceuntre dentro de la nube de partículas.

El siguiente trozo de código ha sido añadido en la función que obtiene el láser ya que si sobrepasaba ese número daba un error. 

![Captura desde 2023-12-26 12-20-37](https://github.com/jlozanot2021/Robotica_Movil_23.24/assets/102520615/b7df7b3b-3fb9-4c9c-96a9-3f6a4647e461)
 
```python
if x >= 0 and x <= 1012 and y >= 0 and y < 1012:
```

### VIDEOS

En los siguientes vídeos se puede apreciar el inicio de las partículas y como se van juntando, como se aproximan al robot y al final, como se solapan con él.

He elegido este vídeo ya que es el que más se aprecía que al inicio están mas dispersas y que aunque parece que no van a lograr juntarse con el robot, por la función propagate_particles y gracias a la aleatoriedad de np.random.normal hace que se acaben juntando.

Parece que todas las partículas salen directamente al rededor del robot, pero esto no es así, lo que pasa es que solo se imprimen por pantalla una vez entrado en el bucle while en el que se sustituyen las que no valen.

(Es el mismo vídeo dividido en trozos porque github no me deja subirlo junto, pese a esto, en localizarse tarda al rededor de 40s con 100 partículas)

[Grabación de pantalla desde 05-01-24 11:56:47.webm](https://github.com/jlozanot2021/Robotica_Movil_23.24/assets/102520615/6f0d5889-183d-4cce-a5d7-be8537cd812e)

[Grabación de pantalla desde 05-01-24 11:57:27.webm](https://github.com/jlozanot2021/Robotica_Movil_23.24/assets/102520615/0c027355-8fc6-4c2f-b993-e5c049316d8e)

[Grabación de pantalla desde 05-01-24 11:57:45.webm](https://github.com/jlozanot2021/Robotica_Movil_23.24/assets/102520615/18a45470-7142-43fa-bfb0-5cb7964934ea)

