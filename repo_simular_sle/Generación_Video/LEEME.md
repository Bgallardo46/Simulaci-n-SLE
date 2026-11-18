# SLE: un atlas de deformaciones conformes

Tres versiones para κ = 2, 6 y 20. Cada película muestra una retícula luminosa y rosetones que se deforman, un acercamiento a la curva, un mapa de dilatación y un relieve tridimensional de ese escalar.

**Videos:** `SLE_kappa_2_4K.mp4`, `SLE_kappa_6_4K.mp4`, `SLE_kappa_20_4K.mp4`. Formato MP4/H.264, 3840 × 2160, 24 fps, 30 s, sin audio. La retícula se calcula sobre la malla nativa 4K. El acercamiento posterior amplía ese fotograma y no añade resolución matemática. El relieve utiliza una malla de 151 × 85 muestras.

Para PowerPoint: insertar el MP4 desde el equipo. Los tres videos usan el mismo encuadre, reloj, textura y realización browniana, de modo que pueden reproducirse simultáneamente. El recorte de acercamiento sigue la región visitada por cada curva.

## Qué se representa

Usamos la convención cordal

\[
\partial_t g_t(z)=\frac{2}{g_t(z)-U_t},\qquad U_t=\sqrt{\kappa}\,B_t,\quad 0\leq t\leq1.
\]

El mapa inverso \(f_t=g_t^{-1}\) lleva el semiplano superior al dominio restante. Para dibujar una textura transportada por \(f_t\), se calcula sobre cada píxel físico \(z\):

\[
C_t(z)=C_0(g_t(z)).
\]

Así, la misma posición de la textura original conserva su color al deformarse. Los colores del rosetón identifican posiciones; no son una medida de dilatación.

En la segunda representación los colores sí codifican el cambio local de escala:

\[
S_t(z)=\log_2|f_t'(g_t(z))|=-\log_2|g_t'(z)|.
\]

Azul indica contracción; naranja, expansión. \(S=1\) significa multiplicar longitudes por 2 y áreas por 4. La escala de color y la altura se saturan en [−3,3] para mostrar cambios de hasta un factor 8 y su inverso. Las colas no desaparecen del cálculo: se recortan únicamente en esta representación. La altura del relieve representa \(S_t\), **no una tercera dimensión de la traza SLE**.

El tiempo de simulación es la capacidad de Loewner. Durante el crecimiento se muestra linealmente desde 0 hasta 1; el video después mantiene t = 1 para el acercamiento y el relieve. No hay un suavizado temporal del browniano.

## Discretización y comparación

El browniano maestro tiene semilla 23 y 262.144 incrementos gaussianos independientes de varianza \(1/262144\). Los tres conductores se obtienen multiplicándolo por \(\sqrt{\kappa}\). Las mallas más gruesas son subconjuntos anidados del maestro: al refinar se conserva la misma realización. No se aplica valor absoluto, recorte, filtrado ni selección de incrementos.

Este módulo utiliza **rendijas verticales con conductor constante al extremo derecho de cada intervalo**, una elección explícita que puede diferir de la discretización de otros notebooks del proyecto. Los mapas elementales y sus inversos son

\[
G_j(z)=U_j+\sqrt{(z-U_j)^2+4\Delta t},\qquad
F_j(w)=U_j+\sqrt{(w-U_j)^2-4\Delta t}.
\]

Se elige la rama adecuada en el semiplano superior. Se componen los mapas, se propaga analíticamente su derivada y se reconstruyen las puntas por composición inversa. Los cálculos de mapas y derivadas usan precisión doble; solo el sombreado de colores usa precisión simple.

| κ | Pasos N | Δt | P95 de diferencia en f₁ | P95 de diferencia en log₂\|f₁′\| |
|---:|---:|---:|---:|---:|
| 2 | 32.768 | 3,05176 × 10⁻⁵ | 2,6211 × 10⁻⁵ | 5,5233 × 10⁻⁴ |
| 6 | 65.536 | 1,52588 × 10⁻⁵ | 4,5251 × 10⁻⁵ | 1,2141 × 10⁻³ |
| 20 | 131.072 | 7,62939 × 10⁻⁶ | 8,1568 × 10⁻⁵ | 2,9028 × 10⁻³ |

La tabla compara con N = 262.144 en 1.024 puntos del semiplano fuente, con Re w entre −7 y 3 e Im w entre 0,02 y 5, al tiempo final. La derivada inversa se estima en este diagnóstico mediante diferencias centradas de paso 10⁻⁵. Las diferencias de posición están en unidades del plano complejo; el píxel de la vista general mide 0,0021875 unidades. Se incluye otra calibración sobre 512 puntos y cinco niveles de refinamiento.

Son **diagnósticos sobre muestras finitas**, no cotas globales ni garantías de convergencia de la traza o de sus detalles próximos a la frontera. Tampoco comparan campos en coordenadas físicas idénticas: comparan mapas y derivadas en los mismos puntos fuente.

Se verificaron el mapa y la derivada GPU frente a una implementación NumPy, el caso exacto U = 0, la simetría ante U ↦ −U y la fórmula exacta del mapa inverso de la rendija vertical. Los errores de referencia quedaron entre aproximadamente 10⁻¹⁶ y 4 × 10⁻¹³.

## Qué detalle no está resuelto

Una aproximación con conductor a saltos puede tener saltos entre puntas. El muestreo de la traza se refina adaptativamente dentro de cada malla temporal. En la vista general **no se dibujan segmentos rectos entre puntas separadas por más de 8 píxeles 4K**; en esos lugares se dibujan puntos. Hay 2, 315 y 4.629 separaciones que superan ese umbral para κ = 2, 6 y 20. Esto explica la apariencia granular de algunas zonas y evita puentes artificiales. El acercamiento amplía también esos puntos.

El umbral Im gₜ(z) ≤ 10⁻¹² detiene el cálculo del píxel y lo oscurece. Es una máscara numérica próxima a la frontera, no una identificación exacta del hull. El filtrado de textura reduce frecuencias que no puede resolver un píxel. El relieve conecta muestras de una malla gruesa y debe interpretarse como una vista cualitativa de S, especialmente cerca de la frontera.

La versión κ = 20 tiene muchos detalles a escala menor que el píxel; el video no certifica propiedades límite como llenar el espacio. Aumentar N o la resolución del video no sustituye un análisis matemático de convergencia en esa frontera.

## Uso y reproducción

Descomprimir el paquete completo en una carpeta y abrir `SLE_Atlas.ipynb`. Las películas se pueden reproducir sin Python ni GPU.

Para recalcular se necesita Python, una GPU NVIDIA compatible con PTX `sm_75` o posterior, controlador CUDA y CuPy. Se probó con una RTX 2070 Max-Q de 8 GB y Python 3.13. No se necesita un compilador CUDA local: los kernels PTX están incluidos. `requirements.txt` registra las versiones utilizadas. FFmpeg se obtiene desde `imageio-ffmpeg`; el codificador predeterminado es NVENC.

```text
python -m pip install -r requirements.txt
python calibrar.py
python verificar_campo.py
python renderizar.py --preview
python renderizar.py
```

Si NVENC no está disponible: `python renderizar.py --codec libx264`. Las versiones existentes se conservan; `--force` vuelve a renderizarlas. Se puede seleccionar, por ejemplo, `--kappa 6`. Las pruebas son videos de 6 s y 960 × 540; los finales son de 30 s y 3840 × 2160. Una interrupción conserva los videos terminados, pero la versión que esté incompleta se recalcula desde t = 0 al volver a ejecutar.

`calibracion.json` y `calibracion_campo.json` contienen las mediciones. Los JSON asociados a cada MP4 registran resolución, duración, tiempo de campo, tiempo total, tamaño y diagnóstico de la traza. `tiempos_finales.json` reúne los resultados finales, si ya están disponibles.

El archivo `browniano_semilla23_2pow18.npz` guarda la realización original. Las cachés `traza_k*.npz` y `campo_k*.npz` permiten inspeccionar los datos sin volver a simular. El notebook y los módulos deben permanecer en la misma carpeta. Para cambiar la realización hay que actualizar la configuración del generador y regenerar las cachés; cambiar solo una semilla en un archivo de salida no modifica los datos guardados.

## Archivos de código

- `sle_gpu.py`: mapas, derivadas, reconstrucción de puntas, comparación NumPy.
- `sle_shader.py`: textura y coloreado de la dilatación.
- `preparar_trazas.py`: conductor compartido y muestreo de puntas.
- `calibrar.py` y `verificar_campo.py`: calibración numérica y rendimiento.
- `renderizar.py`: montaje, anotaciones, codificación MP4 y relieve.

Referencias de contexto: T. Kennedy, [A fast algorithm for simulating the chordal Schramm–Loewner evolution](https://arxiv.org/abs/math/0508002); [Numerical computations for the Schramm–Loewner evolution](https://arxiv.org/abs/0909.2438). El código incluido no implementa la aceleración por series por bloques de esos trabajos: utiliza composiciones elementales en paralelo sobre los píxeles.
