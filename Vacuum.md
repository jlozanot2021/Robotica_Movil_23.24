Al comenzar el proyecto lo primero que hice fue probar los diferentes comandos que nos daban para enteder como funcionaban y que podía hacer con la aspiradora. 
Al entender las funciones que me proporcionaban empece a probar como hacer la primera maquina de estados.
La primera maquina de estados se basaba en tres estados: espiral, retroceder y girar y seguir recto. De espiral se pasaba a ir atrás y girar si se chocaba en algún momento.
Despues esto pasaría a ir recto un tiempo aleatorio y de vuelta a la espiral.
Al principio comence poniendo unos numeros determinados los cuales yo habia pensado y tras probar unos cuantos opte por poner algunos valores de manera aleatria.
Esto daría más posibilidades de completar el objetivo ya que al repetirse a vece se quedaba medio pillado.
Aunque no limpia el 100% de las "habitaciones", si consige un gran trabajo ya que al realizar espirales y luego seguir recto de manera durante un tiempo aleatorio puede llegar a sitios que hagan que cubra bastante espacio.
![Screenshot from 2023-09-24 18-23-23](https://github.com/jlozanot2021/Robotica_Movil_23.24/assets/102520615/f3fa9c72-09f3-4e66-8352-3f79fbd9d4c1)


Fui poniendo más y más variables aleatoras las cuales hacían más impredecible lo que iba a pasar hasta que se me ocurrió poner el lado al que giraría tambien aleatorio según si el numero que saldría era par o impar.
de esta manera podria entrar en algunos lugares que en la prueba de antes no conseguí llegar
```python
      if (random.randint(0 , 1) % 2 == 0):
        left_or_right = 1
      else:
        left_or_right = -1
```

Tras ver que la gran parte del tiempo el robot iba bien, opte por cambiar los time.sleep ya que habia a veces que se chocabay no hacía nada por evitarlo.
De esta manera, al hacer alguna función si se choca volverá a ir atrás y hará el giro.
```python
def clock(state_actual, time_stop,random_time):
  actual_time = time.time()
  if ((actual_time - time_stop) >= random_time):
    return 0
  if (HAL.getBumperData().state == 1):
    return 1
  return state_actual
```


Al final he acabado añadiendo separando los estados atras y giro ya que prefiero que esto lo haga por partes.
Queda así de momento: ESTADOS: espiral (si se choca)-> atras -> giro -> hacia delante (que puede ir atras si se choca o espiral de nuevo si termina el temporizador aleatorio) 

Como última implementación, le he metido que tenga la posibilidad de pasar de espiral a hacia delante (es muy raro que ocurra).
```python
      if (random.uniform(0 , 1) >= 0.9990):
        state = 3
        actual_time = time.time()
        random_time = random.uniform(1, 3)
``
Al final del todo, tambien he metido el laser aun que he dejado las funciones anteriores del bumper, por si el laser no leyese bien algun valor y se chocase.
```python
def laser_detectec(laser):
  for i in range(0, 180, 30):
    if i == 180:
      i = 179
    if laser[i][0] < 0.35:
      if i < 60:
        return 2
      elif i >= 60 and i <= 120:
        return 1
      elif i > 120:
        return 0
  return -1
```

Este es el resultado final
[Screencast from 09-28-2023 11:20:53 AM.zip](https://github.com/jlozanot2021/Robotica_Movil_23.24/files/12747869/Screencast.from.09-28-2023.11.20.53.AM.zip)
