## 1. ¿En qué se diferencian los pesos y los sesgos?

Los pesos (weights) y los sesgos (biases) son parámetros que utiliza una red neuronal para tomar decisiones y aprender a partir de los datos. Los pesos indican qué tan importante es cada dato y los sesgos permiten ajustar la decisión de la red neuronal.

- Pesos: determinan qué tanta importancia tiene cada dato de entrada para tomar una decisión. Por ejemplo, si la energía de la mascota es baja, ese dato podría tener un peso importante para identificar el estado de cansancio.
- Sesgos: son valores adicionales que permiten ajustar el resultado de una neurona, independientemente de las entradas. Ayudan a que la red pueda modificar el punto a partir del cual se activa una respuesta.

Por ejemplo, una neurona podría realizar esta operación:

\\[ z = (w_1 \cdot energía) + (w_2 \cdot alimentación) + b \\]

En este caso, los pesos determinan la influencia de la energía y la alimentación, mientras que el sesgo permite ajustar el resultado final.

## 2. ¿Por qué el cansancio es `1 - energía`?

Se utiliza la operación `1 - energía` porque se considera que el cansancio es inversamente proporcional a la energía: cuando la energía disminuye, el cansancio aumenta, y viceversa.

Para entenderlo, podemos utilizar valores entre 0 y 1:

| Energía     | Cansancio (`1 - energía`) |
| ----------- | ------------------------- |
| 1.0 (100 %) | 0.0 (0 %)                 |
| 0.8 (80 %)  | 0.2 (20 %)                |
| 0.5 (50 %)  | 0.5 (50 %)                |
| 0.2 (20 %)  | 0.8 (80 %)                |
| 0.0 (0 %)   | 1.0 (100 %)               |

Esto permite representar el cansancio de una manera sencilla y coherente con el nivel de energía de la mascota.

En conclusión: se resta la energía a 1 porque ambos valores representan estados opuestos. Cuando la mascota tiene mucha energía, está poco cansada; cuando tiene poca energía, está muy cansada.

Nota: es una regla matemática de diseño para el proyecto, no una medida fisiológica real.

## 3. ¿Qué hace ReLU y qué hace Softmax?

Son funciones de activación utilizadas en las redes neuronales, pero cumplen propósitos diferentes.

ReLU (Rectified Linear Unit)

ReLU se utiliza generalmente en las capas internas de la red neuronal. Su función consiste en convertir los valores negativos en cero y conservar los positivos.

Su expresión matemática es:

\\[ ReLU(x) = \max(0,x) \\]

Por ejemplo:

| Entrada | Resultado ReLU |
| ------- | -------------- |
| -5      | 0              |
| -2      | 0              |
| 0       | 0              |
| 3       | 3              |
| 8       | 8              |

Esto ayuda a la red neuronal a aprender relaciones más complejas entre los datos.

Softmax

Softmax se utiliza normalmente en la capa de salida cuando se desea clasificar entre varias categorías. Convierte los resultados de la red en valores que pueden interpretarse como probabilidades y cuya suma es igual a 1 (100 %).

En tu proyecto, podría producir un resultado como este:

| Estado de la mascota | Probabilidad |
| -------------------- | ------------ |
| Aburrido             | 15 %         |
| Cansado              | 70 %         |
| Con hambre           | 15 %         |
| Total                | 100 %        |

En este ejemplo, la red considera que el estado más probable es cansado.

En conclusión: ReLU ayuda a la red a aprender patrones durante el procesamiento, mientras que Softmax permite obtener probabilidades para seleccionar una de las tres categorías.

## 4. ¿Por qué una precisión alta puede ser engañosa?

Una precisión alta puede ser engañosa porque no siempre significa que la red neuronal esté clasificando correctamente todos los estados.

Esto puede suceder cuando existe un desequilibrio en los datos, es decir, cuando una categoría tiene muchos más ejemplos que las otras.

Por ejemplo, imaginemos que entrenamos nuestra mascota virtual con 1.000 registros:

| Estado     | Cantidad de registros |
| ---------- | --------------------- |
| Aburrido   | 900                   |
| Cansado    | 50                    |
| Con hambre | 50                    |
| Total      | 1.000                 |

Supongamos que la red neuronal siempre predice que la mascota está aburrida. Aunque no reconoce correctamente los estados de cansancio y hambre, acertaría en 900 de los 1.000 registros.

Esto significa que tendría una exactitud del 90 %, pero realmente estaría fallando en la identificación de dos de los tres estados.

Para evitar este problema, es importante evaluar otras métricas:

- Precisión (Precision): indica qué proporción de las predicciones de una categoría realmente pertenece a esa categoría.
- Recall (Exhaustividad): indica cuántos de los casos reales de una categoría consigue identificar la red.
- F1-score: combina precisión y recall para evaluar el equilibrio entre ambas métricas.
- Matriz de confusión: permite observar qué estados está clasificando correctamente y cuáles está confundiendo.

En conclusión: una precisión o exactitud elevada no garantiza que el modelo funcione correctamente. Es necesario comprobar que pueda reconocer adecuadamente los tres estados de la mascota y no solamente la categoría que aparece con mayor frecuencia.

## Resumen para recordar durante la exposición

| Concepto       | Idea principal                                                     |
| -------------- | ------------------------------------------------------------------ |
| Pesos          | Determinan la importancia de cada entrada.                         |
| Sesgos         | Ajustan el resultado de las neuronas.                              |
| `1 - energía`  | Representa el cansancio como el inverso de la energía.             |
| ReLU           | Convierte valores negativos en cero y conserva los positivos.      |
| Softmax        | Convierte las salidas en probabilidades para clasificar.           |
| Precisión alta | No garantiza un buen modelo si existen categorías desequilibradas. |

Una idea importante para defender tu proyecto: la red neuronal no simplemente memoriza los estados de la mascota, sino que aprende patrones a partir de sus variables de entrada para determinar cuál de los tres estados es el más probable.