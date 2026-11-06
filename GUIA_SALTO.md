# Neurito 2.0: aprende a saltar obstáculos

Neurito aprende **cuándo saltar** ante obstáculos que se aproximan con diferentes velocidades. El cerebro se implementa manualmente con NumPy y el aprendizaje puede observarse en Pygame.

## ¿Qué aprende?

La red recibe tres características normalizadas: distancia al obstáculo, velocidad y tiempo estimado hasta alcanzarlo (distancia / velocidad). La última es una característica derivada, no una observación independiente.

La arquitectura es **3 → 12 → 1**: doce neuronas ocultas con ReLU y una salida con sigmoide. La última devuelve una puntuación entre cero y uno que, con el umbral 0.5, determina si Neurito debe saltar cuando está en el suelo.

## ¿Cómo aprende?

Aplicamos **aprendizaje supervisado**, no aprendizaje por refuerzo. Creamos datos sintéticos que incluyen ejemplos donde es conveniente saltar y otros donde debe esperar. Para generar las etiquetas, definimos una ventana de referencia: saltar cuando el tiempo estimado está entre **7 y 13 fotogramas**. No es una regla aprendida por recompensa ni garantiza evitar choques en todas las situaciones.

La red ajusta sus pesos a partir de la **entropía cruzada binaria**, la **retropropagación** y el **descenso de gradiente**:

`W_nuevo = W_anterior - tasa_de_aprendizaje * gradiente`

El conjunto de entrenamiento equilibra aproximadamente las dos decisiones. De otra manera, una red podría aparentar buena precisión diciendo casi siempre «esperar».

La precisión se mide sobre ejemplos distintos de los utilizados en el entrenamiento. **Una precisión alta no garantiza un buen resultado en el juego**, que también depende de la física del salto.

## Archivos

```text
Neurito/
├── main.py                 # Mascota original: comer, dormir, jugar
├── red-sencilla.py         # Ejemplo de una neurona
├── salto_modelo.py         # Red neuronal manual y datos de entrenamiento
├── salto_juego.py          # Física y detección de colisiones
├── saltar.py               # Juego y visualización del entrenamiento
├── GUIA_SALTO.md
└── assets/
    └── jugar_1.png        # Reutilizado en el nuevo juego
```

## Ejecución

```bash
uv sync
uv run saltar.py
```

No se necesitan dependencias nuevas: se reutilizan NumPy y Pygame.

## Controles

| Tecla | Función |
|---|---|
| `1` | Entrenar una época |
| `2` | Entrenar cien épocas |
| `3` | Activar o pausar entrenamiento continuo |
| `A` | Alternar control automático y manual |
| `Espacio` | Saltar manualmente |
| `C` | Reiniciar pista y contadores |
| `R` | Reiniciar pesos y contadores |
| `Esc` | Salir |

## Experimentos

1. Prueba el juego sin entrenar. Anota colisiones y obstáculos esquivados.
colisiones: siempre y obstaculos esquivados: 0
2. Entrena de cien en cien épocas; compara la pérdida, la precisión y el comportamiento.
100 epocas: perdida de 0.2601, precisión de 91.2%, obstaculos esquivados 18 y choques 16
200 epocas: perdida de 0.2455, precisión de 91.7%, obstaculos esquivados 18 y choques 0
300 epocas: perdida de 0.2166, precisión de 92.7%, obstaculos esquivados 18 y choques 0
400 epocas: perdida de 0.1893, precisión de 92.7%, obstaculos esquivados 18 y choques 0
500 epocas: perdida de 0.2026, precisión de 92.8%, obstaculos esquivados 18 y choques 0
600 epocas: perdida de 0.1550, precisión de 92.8%, obstaculos esquivados 18 y choques 0
700 epocas: perdida de 0.0676, precisión de 97.5%, obstaculos esquivados 18 y choques 0
800 epocas: perdida de 0.1392, precisión de 91.3%, obstaculos esquivados 18 y choques 0
900 epocas: perdida de 0.1572, precisión de 96.7%, obstaculos esquivados 18 y choques 0
1000 epocas: perdida de 0.0496, precisión de 98.1%, obstaculos esquivados 18 y choques 0
3. Cambia `ocultas=12` por `ocultas=3` o `ocultas=24`; compara generalización.
3 ocultas y 200 epocas: perdida de 0.2356, precisión de 91.9%, obstaculos esquivados 70 y choques 4
24 ocultas y 200 epocas: perdida de 0.2739, precisión de 91.5%, obstaculos esquivados 70 y choques 5
4. Modifica la tasa de aprendizaje `0.8` a `0.01` o `1.2` y estudia la convergencia.
0.01 y 200 epocas: perdida de 0.5866, precisión de 87.1%, obstaculos esquivados 0 y choques 70
1.2 y 200 epocas: perdida de 0.2448, precisión de 93.0%, obstaculos esquivados 70 y choques 0
5. Cambia la ventana de entrenamiento en `etiqueta()` y en la generación de positivos. Comprueba si la política resultante es compatible con la física.

**Pregunta central:** ¿por qué una red puede clasificar ejemplos con buena precisión y aun así chocar?
Una red neuronal puede clasificar ejemplos con buena precisión y aun así chocar porque clasificar correctamente los datos de entrenamiento no significa necesariamente que pueda tomar buenas decisiones en tiempo real.

La precisión mide qué tan bien reconoce los ejemplos utilizados para evaluarla, pero no garantiza que sus decisiones sean compatibles con la física del juego.

Por ejemplo, la red puede aprender correctamente cuándo debe saltar según los datos recibidos, pero puede equivocarse al calcular el momento adecuado para hacerlo. También puede encontrarse con situaciones que no aparecieron durante el entrenamiento o aprender patrones que funcionan con los ejemplos, pero no con las condiciones reales del juego.

Esto puede suceder por diferentes razones: una cantidad insuficiente de datos, una mala configuración de la red, una tasa de aprendizaje inadecuada o una diferencia entre las condiciones utilizadas para entrenarla y las condiciones reales de los obstáculos.

En conclusión: una buena clasificación no garantiza un buen comportamiento. Para que la mascota pueda esquivar obstáculos correctamente, la red no solamente debe aprender a reconocer los ejemplos, sino también tomar decisiones que sean compatibles con el tiempo de reacción y las condiciones físicas del juego.
