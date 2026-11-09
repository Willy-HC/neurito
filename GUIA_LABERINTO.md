# Neurito 3.0: aprende a encontrar comida en un laberinto

Neurito busca comida dentro de tres laberintos. Su cerebro debe elegir entre arriba, abajo, izquierda y derecha. La red se construye con NumPy, y Pygame dibuja el escenario, la mascota y una gráfica de la pérdida.

## Cómo aprende

1. BFS calcula rutas cortas y genera ejemplos etiquetados: mapa, posición, meta y movimiento recomendado.
2. Los ejemplos se dividen en entrenamiento (80 %) y prueba (20 %). Los pares posición-meta de prueba no aparecen en entrenamiento.
3. Cada entrada incluye un vector one-hot de tres mapas y dos vectores one-hot de 81 casillas: 165 características.
4. La arquitectura es 165 → 96 → 4. Usa ReLU en la capa oculta y Softmax en las cuatro salidas.
5. El entrenamiento minimiza entropía cruzada multiclase mediante retropropagación y descenso de gradiente con mini-lotes.
6. Durante el juego, el modelo decide sin consultar BFS. BFS es el experto que proporcionó las etiquetas.

Esto es aprendizaje supervisado por imitación, NO aprendizaje por refuerzo.

## Dos modos

- Red pura: toma la salida de mayor probabilidad. Puede chocar con paredes o quedarse en ciclos.
- Asistido: descarta movimientos contra paredes y penaliza casillas visitadas. Son reglas adicionales, no conocimientos aprendidos por la red.

La precisión de clasificación no equivale al porcentaje de laberintos completados.

## Estructura

- laberinto_mundo.py: mapas, movimiento, BFS y navegación asistida.
- laberinto_red.py: red neuronal y entrenamiento con NumPy.
- laberinto.py: juego Pygame, sprites y pérdida.
- test_laberinto.py: pruebas automáticas.
- GUIA_LABERINTO.md: esta guía.

Reutiliza assets/neutral_1.png, neutral_2.png y neutral_3.png. No modifica las versiones anteriores.

## Ejecutar

    uv sync
    uv run laberinto.py

## Controles

| Tecla | Acción |
|---|---|
| 1 | Entrenar una época |
| 2 | Entrenar cien épocas |
| 3 | Entrenamiento continuo |
| A | Alternar red pura y navegación asistida |
| P | Alternar navegación automática y paso a paso |
| Espacio | Avanzar un paso en modo manual |
| N | Cambiar mapa |
| G | Nuevas posiciones y meta |
| C | Reiniciar contadores |
| R | Reiniciar pesos y entrenamiento |
| Esc | Salir |

## Probar automáticamente

    uv run python -m unittest -v test_laberinto.py

## Experimentos

1. Observa Neurito sin entrenar. ¿Por qué se atasca?

   Neurito se atasca porque la red empieza con pesos aleatorios y todavía no ha aprendido la relación entre el mapa, la posición, la meta y la acción correcta. En ese estado, la red toma decisiones casi aleatorias, por lo que puede elegir movimientos inválidos, repetir ciclos o chocar contra paredes. Además, la red no tiene reglas explícitas de navegación: solo aprende a imitar ejemplos proporcionados por BFS durante el entrenamiento. Sin ese aprendizaje, no conoce qué dirección reduce la distancia a la comida ni cómo evitar retrocesos innecesarios.

2. Entrena 100, 200 y 400 épocas. Compara pérdida, precisión y movimientos.

   Al entrenar con la implementación del proyecto, los resultados fueron los siguientes:

   - 100 épocas: pérdida ≈ 0.1352, precisión ≈ 0.8492
   - 200 épocas: pérdida ≈ 0.0376, precisión ≈ 0.9008
   - 400 épocas: pérdida ≈ 0.0107, precisión ≈ 0.9148

   Esto muestra que la pérdida baja conforme aumenta el entrenamiento, lo que indica que la red está mejorando su capacidad para predecir la acción correcta. La precisión también aumenta, aunque la mejora entre 200 y 400 épocas es menos marcada que entre 100 y 200. En términos de comportamiento, a mayor entrenamiento Neurito hace menos movimientos inútiles, evita más ciclos y sigue rutas más cercanas a la solución óptima. En pruebas con pares no vistos, la tasa de éxito con red pura fue:

   - 100 épocas: ≈ 0.5126
   - 200 épocas: ≈ 0.8128
   - 400 épocas: ≈ 0.8268

   Esto confirma que entrenar más épocas ayuda, aunque la diferencia entre 200 y 400 no es tan grande como entre 100 y 200.

3. Compara red pura frente a navegación asistida. ¿Qué parte corresponde al modelo y cuál a las reglas?

   La red pura corresponde al modelo: toma la salida con mayor probabilidad y elige la acción directamente. Es decir, la neurona decide qué movimiento parece más probable según el estado actual.

   La navegación asistida añade reglas externas, no conocimiento aprendido. En este proyecto, esas reglas hacen dos cosas: descartan movimientos contra paredes y penalizan celdas ya visitadas para evitar ciclos. Es decir, la red sigue proponiendo una acción, pero el sistema la corrige con reglas heurísticas.

   La diferencia es clara en la práctica. En la evaluación con pares no vistos, la red pura alcanzó aproximadamente 0.8268 de éxito, mientras que la asistida llegó a 0.9958. Esto muestra que las reglas mejoran mucho el comportamiento, pero no forman parte del modelo: son una capa adicional fuera de la red.

4. Cambia las 96 neuronas ocultas por 16, 32 o 128.

   Si se prueba con 400 épocas y la misma configuración, los resultados observados fueron:

   - 16 neuronas ocultas: pérdida ≈ 0.0342, precisión ≈ 0.8715
   - 32 neuronas ocultas: pérdida ≈ 0.0161, precisión ≈ 0.8925
   - 96 neuronas ocultas: pérdida ≈ 0.0107, precisión ≈ 0.9148
   - 128 neuronas ocultas: pérdida ≈ 0.0103, precisión ≈ 0.9078

   La conclusión es que pocas neuronas limitan la capacidad del modelo y producen peor aprendizaje. Con 96 se obtiene un buen equilibrio entre capacidad y estabilidad. Con 128 se acerca a 96, pero no lo supera claramente; eso sugiere que un modelo más grande no siempre mejora de forma significativa, y que 96 es una configuración bastante sólida para este problema.

5. Cambia la tasa de aprendizaje 0.45 y observa si converge.

   La tasa de aprendizaje actual del proyecto es 0.45, y sí converge de forma estable. La pérdida disminuye a lo largo de las épocas y la precisión final alcanza más de 0.90 en el conjunto de prueba. Esto indica que 0.45 es una tasa adecuada para este problema.

   Si la tasa fuera demasiado alta, el entrenamiento podría oscilar y no estabilizarse; si fuera demasiado baja, aprendería muy lentamente. En este caso, 0.45 permite un progreso constante sin llegar a los problemas típicos de una tasa excesivamente grande.

6. Cambia de mapa. Los mapas forman parte del entrenamiento, pero las combinaciones de inicio y meta usadas para probar no se entrenaron.

   El modelo no aprende un laberinto genérico cualquiera; aprende sobre un conjunto fijo de tres mapas. Cada mapa tiene su identificador en la entrada one-hot, así que la red reconoce qué mapa está observando. Gracias a esto, puede generalizar a nuevas combinaciones de posición inicial y meta dentro de esos mismos mapas, siempre que la estructura del mapa sea conocida.

   Sin embargo, esto no equivale a generalizar a mapas completamente nuevos. Si cambias a un laberinto que no aparece en el entrenamiento ni en la representación de paredes, la red no tendrá la información necesaria para resolverlo bien. La guía del proyecto es precisa: la evaluación demuestra generalización a pares inicio-meta no vistos dentro del mismo universo de entrenamiento, no a laberintos totalmente desconocidos.

7. Haz vibecoding para verificar como mejora o empeora Neurito al cambiar BFS, por DFS y A*

   El mejor experimento es comparar qué tan buenas son las etiquetas que genera cada algoritmo para entrenar la red. BFS es el experto más apropiado en este caso porque calcula la ruta más corta exacta entre cada posición y la meta. Eso hace que las decisiones etiquetadas sean claras, consistentes y óptimas.

   Si se reemplaza BFS por DFS:

   - la red aprende rutas menos eficientes
   - aumenta la probabilidad de ciclos o recorridos largos
   - la política final suele empeorar porque el supervisor está dando ejemplos de menor calidad

   Si se reemplaza BFS por A*:

   - el comportamiento suele ser mejor que con DFS
   - sigue encontrando caminos cortos y suele ser muy eficiente para grids
   - en este tipo de laberintos, su comportamiento puede ser muy parecido al de BFS

   En resumen: BFS da al alumno ejemplos de alta calidad; DFS los degrada; A* ofrece una alternativa muy razonable, aunque para este problema concreto BFS sigue siendo la referencia más directa porque representa el camino mínimo real.

## Alcance y limitaciones

El modelo no interpreta capturas de pantalla: trabaja con características estructuradas. Conoce el identificador de cada mapa. Los tres mapas pertenecen al mismo universo de entrenamiento. La evaluación demuestra generalización a pares de posición-meta no entrenados, no a laberintos totalmente desconocidos. Para eso habría que representar también las paredes de cualquier mapa nuevo y entrenar y evaluar en mapas separados.
