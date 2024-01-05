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



