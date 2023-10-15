### Follow line

Objetivo:
El objetivo de esta práctica es que el vehículo haga el circuito correspondiente en el menor tiempo posible saliendose lo mínimo de la línea roja.

Visión de la línea:
Para visionar la línea roja que se encuentra sobre el centro del circuito he usado una máscara la cual dibid en bits 1 o 0 según si el pixel se encuentra en el rango indicado (rojo en este caso).
```python
    lowest_thresh = np.array([0, 43, 46])
    top_thresh = np.array([26,255,255])


    hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)

    blur = cv2.GaussianBlur(hsv, (5,5), 0)

    res = cv2.inRange(blur, lowest_thresh, top_thresh)

    h, w, d = image.shape
```
Después de esto,he sacado el momento de esta imagen. Esto significa que me da el punto medio de todos los pixeles que han quedado que son rojos. También printeo el centroide para que se pueda ver.
```python
M = cv2.moments(res)
if M['m00'] > 0:
      cx = int(M['m10']/M['m00'])
      cy = int(M['m01']/M['m00'])
      cv2.circle(image,(cx,cy),5,(255, 0, 128),-1)
```
También añadí un PID el cual al comienzo solo era angular y lo intente estabilizar a velocidad 4.
```python
Kp_c = 1
Ki_c = 0.0001
Kd_c = 6

      cur_erro = -(cx - w/2)/300
      angular = Kp_c*(cur_erro) + Ki_c* err+ Kd_c*(cur_erro -  prev_error)
      prev_error = cur_erro
      err = err + cur_erro

      HAL.setV(lineal)
      HAL.setW(angular)
```
Como solo con esto lo realizaba en demasiado tiempo, decidi intentar diferentes opciones, las cuales relatare a continuación y el porque de su descarte, hasta quedarme con la opción final y mas óptima.

La primera opción fue restar la velocidad angular a un valor fijo de velocidad lineal y así, si estaba muy desviado iría mas lento y si estaba muy centrado, más rápido.
Esta opción fue descartada ya que si se alejaba mucho del centro iría muy lento y esto haría que al volver hacia la recta, subiese de velocidad, resultando en salirse otra vez de la línea.
Aunque al final se conseguía estabilizar, la cantidad de oscilaciones era muy elevada y por tanto subía mucho el tiempo.

La segunda opción consisitío en meter un PID para la velocidad lineal. Creo que esta opción era la idonea para este proyecto, pero el problema fue que no pude encontrar los números requeridos para ello.
Las posibilidades son muchas para la combinación de números en las K, pero las soluciones óptimas son muy pocas. A pesar de que fui regulando poco a poco cada uno de los números, no llegué a dar con unos valores los cuales ayudasen a regular las óscilaciones.
El problema principal fué parecido al anterior, pero este fué mucho más agrabado ya que tardaba mucho mas en estabilizarzse y oscilaba muy brusco.

La última opción y la final, fue meter dos momentos, uno mas arriba y otro mas abajo, para así mirar si lo siguiente qeu venía era una curva o estabamos en una recta.
```python
M2 = cv2.moments(res[0:(h*21)//40, :])
        cx2 = int(M2['m10']/M2['m00'])
        cy2 = int(M2['m01']/M2['m00'])
```
Con esto ya fui probando diferentes formas de como conseguir el objetivo y al final me decante por unirlas todas en una sola.
Por un lado, se detecta si los dos puntos estan muy separados el uno del otro. Esto es muy útil para estimar si es una recta, ya que si el lejano esta alineado con el cercano es que es una línea recta y por tanto, podemos ir a maxima velocidad
```python
if abs(cx - cx2)//10 < 1:
        print("maximo")
        lineal = 10
```
Y por el otro lado, se comprovaba si los puntos estaban muy cerca del centro de la imagen, para saber asi al 100% si era una recta
```python
if (abs(cx - w // 2) <= 5 or abs(cx2 - w // 2) <= 5):
        print("maximo")
        lineal = 10
```
Si los dos puntos estaban muy cerca del centro pero no tanto como una recta, he añadido una funcion la cual hace que el coche vaya acelerando hasta un limite
```python
        if abs(cx - w // 2) <= 25 or abs(cx2 - w // 2) <= 25:
          print("acelerando")
          acelerar += 0.003
          lineal = acelerar
          if acelerar >= 7:
            acelerar = 7
```
Tras todo esto, como a veces mientras se esta estabilizando el coche se pone recto por un u¡instante y no debería coger esa maxima velocidas, ya que no esta estabilizado del todo, decidí poner una array y hacer una media de ella para asegurarme de que podia poner esa máxima velocidad.
```python
velocidades.append(lineal)
      contador += 1
      if contador == 20:
        velocidades.pop(0)
        contador -= 1
       
      media = sum(velocidades) / len(velocidades)
      lineal = 4
      if media == 10:
        lineal = 10
      elif media > 4.1:
        lineal = acelerar
      else:
        lienal = 4
```

Por último he añadido que si no detecta el centroide más lejano, es decir es una curva muy pronunciada, vaya a 0.5
Como se puede observar por los números, me he decantado por la precisión y robustez en vez de la velocidad.

Con todo esto, el coche, en el primer circuito, realiza una vuelta alrededor de los 120s
