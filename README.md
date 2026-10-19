# Taller de ordenamiento: comparación empírica de 5 algoritmos en Python

## 1. Título y objetivo

Trabajo de **Estructura de Datos** (Ingeniería en Ciencia de Datos, UIS). El objetivo es
**implementar a mano** cinco algoritmos de ordenamiento y **compararlos empíricamente**
sobre muestras de una misma lista de 200.000 enteros, midiendo tiempos, graficándolos y
ajustando la función que mejor describe a cada uno.

Algoritmos comparados: **inserción**, **burbuja**, **selección**, **merge sort** y
**quicksort**.

## 2. Algoritmos implementados a mano

Todo el ordenamiento está escrito en el notebook con bucles, comparaciones e intercambios.
**No se usa ninguna función ni librería que ordene**: `sorted()`, `list.sort()`, `heapq`,
`bisect`/`insort`, `numpy.sort`/`argsort`, `pandas.sort_values`/`sort_index`,
`sortedcontainers`, etc. La mezcla de merge sort también se escribe a mano (sin
`heapq.merge`).

- **Inserción**, **Burbuja** y **Selección**: O(n²).
- **Merge sort** (recursivo, con mezcla propia) y **Quicksort** (pivote aleatorio,
  iterativo con pila explícita para no desbordar el límite de recursión): O(n log n).

La validación se hace con `assert`, comparando con `Counter` y una función `es_ordenada`
propia, sin usarse `sorted()`, antes del benchmark.

## 3. Estructura del proyecto

```
.
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   └── numeros.txt              # 200.000 enteros (se regenera con semilla 42, no versionado)
├── results/
│   ├── tiempos.csv              # todas las mediciones (algoritmo, n, repetición, tiempo)
│   ├── promedios.csv            # promedio y desviación por algoritmo y tamaño
│   ├── tiempos_lineal.png
│   ├── tiempos_loglog.png
│   ├── tiempos_merge_quick.png
│   └── ajustes.png
└── notebooks/
    └── taller_ordenamiento.ipynb   # notebook principal
```

## 4. Cómo reproducir (Debian + VS Code)

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m ipykernel install --user --name sorting-benchmark --display-name "Python (sorting-benchmark)"
```

En VS Code: abre `notebooks/taller_ordenamiento.ipynb` y selecciona el kernel
**Python (sorting-benchmark)** (el del `.venv`).

Ejecución:

1. Con `QUICK_MODE = True` (valor por defecto) el notebook corre pocos tamaños y 2
   repeticiones en minutos, para validar el flujo.
2. Para el experimento final, cambia a `QUICK_MODE = False` y ejecuta todo (*Run All*).

También se puede ejecutar sin abrir VS Code:

```bash
jupyter nbconvert --to notebook --execute --inplace \
  --ExecutePreprocessor.timeout=-1 \
  --ExecutePreprocessor.kernel_name=sorting-benchmark \
  notebooks/taller_ordenamiento.ipynb
```

## 5. Metodología

- **Datos.** Se genera `data/numeros.txt` con 200.000 enteros aleatorios y semilla fija
  (`random.seed(42)`). Es reproducible y no se versiona.
- **Muestras.** Todas son **prefijos** de la misma lista, con tamaños
  **500, 1.000, 10.000, 30.000, 50.000, 100.000 y 200.000**.
  - Nota: el enunciado dice *"cuentas mil"*; se interpreta como **50.000**.
  - La cantidad fue reducida por el profesor de 500.000 a **200.000** enteros.
- **Medición.** Por cada combinación (algoritmo, tamaño) se ordena una **copia**
  (`list(muestra)`) y se mide con `time.perf_counter()`. Se repite **5 veces** cada
  combinación y se promedian los tiempos.
- **Benchmark dividido por tamaño.** El notebook tiene una celda por cada tamaño exigido
  (quinientos, mil, diez mil, treinta mil, cincuenta mil, cien mil, doscientos mil), en
  ese orden.
- **Límite de tiempo para los O(n²).** En Python puro, burbuja/selección/inserción con n
  grande tardan decenas de minutos por corrida. Antes del benchmark se mide cada algoritmo
  con n = 2.000 y se extrapola por escala n² (y n·log n para merge/quick). Si la
  estimación de una corrida supera `MAX_SEGUNDOS_POR_CORRIDA` (300 s) o si `n` supera
  `N_MAX_CUADRATICOS`, ese tamaño se omite para ese algoritmo: queda como `NaN` en la
  tabla y se avisa por pantalla. Merge sort y quicksort corren todos los tamaños.
- **Guardado parcial.** Cada combinación se escribe en `results/tiempos.csv` al terminar;
  el avance en vivo queda en `results/progreso.log` (útil con `tail -f`).

En la corrida publicada en `results/` se usó `N_MAX_CUADRATICOS = 30.000`, por lo que los
cuadráticos se corrieron hasta n = 30.000; para incluir 50.000/100.000 basta subir ese
tope y volver a ejecutar.

## 6. Resultados

Tiempo **promedio (segundos)** por algoritmo y tamaño (`results/promedios.csv`):

| algoritmo | 500 | 1.000 | 10.000 | 30.000 | 50.000 | 100.000 | 200.000 |
|---|---|---|---|---|---|---|---|
| Inserción | 0.0039 | 0.0176 | 2.3203 | 20.4506 | – | – | – |
| Burbuja | 0.0099 | 0.0409 | 4.5645 | 47.8461 | – | – | – |
| Selección | 0.0047 | 0.0191 | 1.8242 | 18.6374 | – | – | – |
| Merge sort | 0.0007 | 0.0015 | 0.0276 | 0.0745 | 0.1162 | 0.2690 | 0.5404 |
| Quicksort | 0.0010 | 0.0018 | 0.0370 | 0.0519 | 0.0964 | 0.2015 | 0.4149 |

Ajuste de curvas (`curve_fit`) y pendiente en escala log-log:

| algoritmo | mejor modelo | R² | RMSE | pendiente log-log |
|---|---|---|---|---|
| Inserción | `a·n² + b` | 0.999994 | 0.0216 | 2.10 |
| Burbuja | `a·n² + b` | 0.999757 | 0.3139 | 2.07 |
| Selección | `a·n² + b` | 0.999825 | 0.1035 | 2.01 |
| Merge sort | `a·n·log(n) + b` | 0.999058 | 0.0056 | 1.12 |
| Quicksort | `a·n·log(n) + b` | 0.997423 | 0.0070 | 1.00 |

Gráficas generadas en `results/`:

![Tiempo promedio vs n (escala lineal)](results/tiempos_lineal.png)

![Tiempo promedio vs n (escala log-log)](results/tiempos_loglog.png)

![Merge sort vs Quicksort](results/tiempos_merge_quick.png)

![Datos vs mejor modelo ajustado](results/ajustes.png)

## 7. Conclusiones

- **La complejidad empírica coincide con la teórica.** Los tres cuadráticos se ajustan a
  `a·n² + b` (R² ≥ 0.9997) con pendiente log-log ≈2; merge sort y quicksort se ajustan a
  `a·n·log(n) + b` con pendiente ≈1,0–1,1.
- **Las constantes importan.** A n = 30.000, quicksort es ≈390× más rápido que burbuja y
  merge sort ≈270× más rápido que inserción, pese a que "todos ordenan".
- **Burbuja es el peor de los cuadráticos** porque intercambia en cada inversión; inserción
  mueve con asignaciones simples y selección casi solo compara (y en Python puro queda
  empatada con inserción).
- **Escalabilidad.** Extrapolando por n², una corrida de los cuadráticos con 200.000
  elementos tardaría ≈800 s (inserción/selección) y ≈1900 s (burbuja); con 5 repeticiones
  son horas. Merge sort y quicksort ordenan 200.000 en ≈0,5 s.
- **Qué dice la gráfica:** en escala lineal los O(n²) se disparan y aplastan a los
  O(n log n); en **log-log** las pendientes (≈2 vs ≈1) separan las familias y dejan ver las
  constantes (altura) y la escalabilidad. Es la evidencia visual de que la complejidad
  asintótica manda cuando n crece.

## 8. Limitaciones

- Los tiempos son de **un solo equipo** (CPython 3.14) y no están aislados del ruido del
  sistema (GC, planificador); por eso se reportan promedios de 5 repeticiones y su
  desviación.
- Las muestras son **prefijos de la misma lista**; no se promedian muchas listas distintas.
- Los O(n²) se limitaron a n ≤ 30.000 por tiempo; los valores para n mayores son
  **extrapolaciones** por escala n², no mediciones.
- Con 4 puntos (cuadráticos) el ajuste es sensible: se refuerza con la pendiente log-log.
- Merge sort usa memoria auxiliar O(n); quicksort ordena in-place.
- Los tiempos absolutos dependen del hardware; el interés está en las **tendencias**.
