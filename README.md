---

## Universidad de Costa Rica
### Escuela de Ingeniería Eléctrica
#### IE0405 - Modelos Probabilísticos de Señales y Sistemas

Segundo semestre del 2020

---

* Estudiante: **Jorge Daniel Rodríguez Hernández**
* Carné: **A95284**
* Grupo: **2**

# `L5` - *Cadenas de Markov*

La administración del servicio desea ahora que el servidor no esté vacío (sin atender solicitudes) más del 10% del tiempo. 
Hallar el parámetro $\nu$ para satisfacer este requisito y modificar el código para medir la variable de interés en una simulación.

Se importan todas las bibliotecas de python a utilizar

```python
import numpy as np
from scipy import stats
import matplotlib.pyplot as plt
```
se crean las constantes, las variables aleatorias y los vectores con los datos de las variables aleatorias que cumplen con los requerimientos del enunciado.

```python

# Número de clientes
N = 1000

# Parámetro de llegada (clientes/segundos)
lam = 2/60

# Parámetro de servicio (servicios/segundos)
nu = 2.22222222222222222222222222222222222222222222/60

# Distribución de los tiempos de llegada entre cada cliente
X = stats.expon(scale = 1/lam)

# Distribución de los tiempos de servicio a cada cliente
Y = stats.expon(scale = 1/nu)

# Intervalos entre llegadas (segundos desde último cliente)
t_intervalos = np.ceil(X.rvs(N)).astype('int')
```
Se inicializan los tiempos de llegada y se crean los tiempos de servicio para poder manejar los tiempos en que los clientes ingresan y salen del servidor con el 
fin de medir los tiempos de atencion de los clientes.

```python
# Tiempos de las llegadas (segundos desde el inicio)
t_llegadas = [t_intervalos[0]]
for i in range(1, len(t_intervalos)):
    siguiente = t_llegadas[i-1] + t_intervalos[i]
    t_llegadas.append(siguiente)

# Tiempos de servicio (segundos desde inicio de servicio)
t_servicio = np.ceil(Y.rvs(N)).astype('int')

# Inicialización del tiempo de inicio y fin de atención
inicio = t_llegadas[0]          # primera llegada
fin = inicio + t_servicio[0]    # primera salida

# Tiempos en que recibe atención cada i-ésimo cliente (!= que llega)
t_atencion = [inicio]
for i in range(1, N):
    inicio = np.max((t_llegadas[i], fin))
    fin = inicio + t_servicio[i]
    t_atencion.append(inicio)

# Inicialización del vector temporal para registrar eventos
t = np.zeros(t_atencion[-1] + t_servicio[-1] + 1)

# Asignación de eventos de llegada (+1) y salida (-1) de clientes
for c in range(N):
    i = t_llegadas[c]
    t[i] += 1
    j = t_atencion[c] + t_servicio[c]
    t[j] -= 1

```
Se parametriza la cantidad de personas posibles minima en el servidor y dado que se busca que al menos exista una persona dentro segun los requerimientos
solicitados por la administracion el valor de P será de 1 
```python

# Umbral de P o más personas en sistema (hay P - 1 en fila)
P = 1

# Instantes (segundos) de tiempo con P o más solicitudes en sistema
frecuencia = 0

# Proceso aleatorio (estados n = {0, 1, 2...})
Xt = np.zeros(t.shape)

# Inicialización de estado n
n = 0

# Recorrido del vector temporal y conteo de clientes (estado n)
for i, c in enumerate(t):
    n += c # sumar (+1) o restar (-1) al estado
    Xt[i] = n
    if Xt[i] >= P: 
        frecuencia += 1

# Fracción de tiempo con P o más solicitudes en sistema
fraccion = frecuencia / len(t)

# Resultados
print('Parámetro lambda =', str(lam*60))
print('Parámetro nu =', str(nu*60))
print('Tiempo con más de {} solicitudes en fila:'.format(P-2))
print('\t {:0.2f}%'.format(100*fraccion))
if fraccion >= 0.90:
    print('\t Sí cumple con la especificación.')
else:
    print('\t No cumple con la especificación.') 
print('Simulación es equivalente a {:0.2f} horas.'.format(len(t)/3600))

# Gráfica de X(t) (estados del sistema)
plt.figure()
plt.plot(Xt)
plt.plot(range(len(t)), (P-1)*np.ones(t.shape))
plt.legend(('$X(t) = n$', '$L_q = $' + str(P-2)))
plt.ylabel('Clientes en el sistema, $n$')
plt.xlabel('Tiempo, $t$ / segundos')
plt.show()

```
Al utlizarse el codigo anterior se obtiene la siguiente grafica

![90](https://github.com/jorgedaniel-rodriguez/Tema5/blob/main/90.png)

En dicha grafica se observa que los datos obtenidos cumplen con las solicitudes de la administracion, dado que lo que se solicitó fue que 
el servidor no estuviera por mas del 10% sin usuarios, con la posible intencion de aprovechar al maximo los recuersos del servidor sin llegar a un 
sobrecargo de trabajo.

![90](https://github.com/jorgedaniel-rodriguez/Tema5/blob/main/ecuacion.png)

Para cumplir con dicha solicitud y dado que el ingreso de clientes al servidor es de aproximadamente 2 usuarios, se utilizo la ecuacion anterior para encontrar el 
parametro del servicio, siendo este de 2.222222 aproximadamente al esperarse que la probabilidad de que 1 o más usuarios en el servicio no disminuya del 90%; como se
puede observar los parametros escogidos cumplen con lo esperado y se concluye que es la cantidad de usuarios que satisfacen el sistema.

