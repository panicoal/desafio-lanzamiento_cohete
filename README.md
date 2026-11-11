# Desafío de Métodos Numéricos: Misión Espacial NASA 1962 — El Método de Euler y su Extensión a Heun y RK4

> **Contexto:** Inspirado en la historia real retratada en _Talentos Ocultos_ (_Hidden Figures_) y en el trabajo de las matemáticas Katherine Johnson, Dorothy Vaughan y Mary Jackson en el Centro de Investigación de Langley de la NASA durante el vuelo orbital de John Glenn (misión _Friendship 7_, Proyecto Mercury, 1962).

---

## 1. Resumen Ejecutivo del Proyecto

El proyecto consiste en el modelado matemático, simulación interactiva y análisis numérico del ascenso vertical de un cohete experimental durante sus primeros 20 segundos de vuelo.

Frente a las limitaciones de las computadoras electrónicas de la época (IBM 7090), se implementan y contrastan tres esquemas de integración numérica de ecuaciones diferenciales ordinarias (EDO):

1. **Método de Euler** (Primer orden, $\mathcal{O}(h)$)
2. **Método de Heun / Euler Mejorado** (Segundo orden, predictor-corrector, $\mathcal{O}(h^2)$)
3. **Runge-Kutta de Cuarto Orden - RK4** (Cuarto orden, $\mathcal{O}(h^4)$)

Todos los métodos son validados cuantitativamente contra la **solución analítica exacta de Katherine Johnson**, demostrando experimentalmente el orden de convergencia y evaluando el cumplimiento de las tolerancias de seguridad de la misión de la NASA.

---

## 2. Parámetros del Cohete Experimental (NASA 1962)

Los parámetros físicos del modelo corresponden a los datos del desafío didáctico:

| Parámetro                         |   Símbolo    |   Valor   |    Unidades    |
| :-------------------------------- | :----------: | :-------: | :------------: |
| Masa del cohete                   |     $m$      | $1\,000$  |  $\text{kg}$   |
| Empuje del motor                  |     $T$      | $15\,000$ |   $\text{N}$   |
| Aceleración de la gravedad        |     $g$      |  $9.81$   | $\text{m/s}^2$ |
| Coeficiente de resistencia lineal |     $k$      |   $100$   | $\text{kg/s}$  |
| Velocidad inicial                 | $v(0) = v_0$ |    $0$    |  $\text{m/s}$  |
| Altura inicial                    | $y(0) = y_0$ |    $0$    |   $\text{m}$   |
| Tiempo de simulación              |    $t_f$     |   $20$    |   $\text{s}$   |
| Paso base de integración          |     $h$      |   $2.0$   |   $\text{s}$   |

---

## 3. Modelo Físico y Matemático

Aplicando la **Segunda Ley de Newton** en el eje vertical ($F_{\text{neta}} = m \cdot a$):

$$m \frac{dv}{dt} = T - m \cdot g - k \cdot v$$

Dividiendo entre la masa $m$:

$$\frac{dv}{dt} = \left(\frac{T}{m} - g\right) - \left(\frac{k}{m}\right) v$$

Definiendo los coeficientes del sistema:

- $a = \frac{T}{m} - g = \frac{15\,000}{1\,000} - 9.81 = 5.19\text{ m/s}^2$ (aceleración neta inicial sin resistencia).
- $b = \frac{k}{m} = \frac{100}{1\,000} = 0.1\text{ s}^{-1}$ (coeficiente de amortiguamiento lineal).

Puesto que $v(t) = \frac{dy}{dt}$, el sistema acoplado de primer orden es:

$$
\begin{cases}
\dfrac{dv}{dt} = 5.19 - 0.1 \cdot v, & v(0) = 0 \\
\dfrac{dy}{dt} = v, & y(0) = 0
\end{cases}
$$

### Supuestos del modelo y limitaciones físicas:

- **Masa constante:** No considera el consumo de combustible (en un cohete real rige la ecuación de Tsiolkovski, $\frac{dm}{dt} < 0$).
- **Resistencia aerodinámica lineal ($kv$):** Válida a bajas velocidades en régimen laminar; en ascensos reales predomina la resistencia cuadrática ($\frac{1}{2}\rho v^2 C_D A$) y la densidad del aire $\rho(y)$ decae con la altura.
- **Gravedad constante ($g$):** Asume $g = 9.81\text{ m/s}^2$ invariable en los primeros 600 metros.

---

## 4. Solución Analítica Exacta

### Deducción de la Velocidad $v(t)$

Por separación de variables:

$$\frac{dv}{5.19 - 0.1 \cdot v} = dt \implies -10 \ln|5.19 - 0.1 \cdot v| = t + C_1$$

Despejando y aplicando la función exponencial:

$$5.19 - 0.1 \cdot v = C \cdot e^{-0.1 \cdot t} \implies v(t) = 51.9 - 10 \cdot C \cdot e^{-0.1 \cdot t}$$

Con la condición inicial $v(0) = 0$:

$$0 = 51.9 - 10 \cdot C \implies C = 5.19$$

$$\mathbf{v(t) = 51.9 \cdot \left(1 - e^{-0.1 \cdot t}\right)}$$

_Velocidad límite teórica:_ $v_\infty = \lim_{t \to \infty} v(t) = \frac{a}{b} = \mathbf{51.9000\text{ m/s}}$.

---

### Deducción de la Altura $y(t)$

Integrando la velocidad respecto al tiempo:

$$y(t) = \int v(t) \, dt = \int 51.9 \cdot \left(1 - e^{-0.1 \cdot t}\right) dt = 51.9 \cdot t + 519 \cdot e^{-0.1 \cdot t} + C_2$$

Con la condición inicial $y(0) = 0$:

$$0 = 51.9(0) + 519 \cdot e^0 + C_2 \implies C_2 = -519$$

$$\mathbf{y(t) = 51.9 \cdot t - 519 \cdot \left(1 - e^{-0.1 \cdot t}\right)}$$

---

### Evaluación Exacta a los 20 Segundos ($t = 20\text{ s}$)

Con $e^{-0.1(20)} = e^{-2} \approx 0.13533528$:

- **Velocidad exacta:** $v(20) = 51.9 \cdot (1 - e^{-2}) = \mathbf{44.8761\text{ m/s}}$
- **Altura exacta:** $y(20) = 51.9(20) - 519 \cdot (1 - e^{-2}) = 1038 - 448.7610 = \mathbf{589.2390\text{ m}}$

---

## 5. Algoritmos de los Métodos Numéricos

### A. Método de Euler (Primer Orden, $\mathcal{O}(h)$)

Evalúa la pendiente una sola vez al inicio del intervalo:
$$v_{n+1} = v_n + h \cdot (5.19 - 0.1 \cdot v_n)$$
$$y_{n+1} = y_n + h \cdot v_n$$

_Comportamiento físico:_

- **Sobreestima la velocidad:** Usa la aceleración inicial de cada paso, cuando la velocidad es menor y por ende la resistencia $kv$ es menor a la real.
- **Subestima la altura:** Usa la velocidad del inicio del paso ($y_{n+1} = y_n + h \cdot v_n$), ignorando que el vehículo estuvo acelerando continuamente durante el intervalo.

---

### B. Método de Heun / Predictor-Corrector (Segundo Orden, $\mathcal{O}(h^2)$)

1. **Pendientes iniciales en $t_n$:**
   $$k_{1,v} = 5.19 - 0.1 \cdot v_n, \quad k_{1,y} = v_n$$
2. **Paso Predictor (Euler hacia adelante):**
   $$\tilde{v}_{n+1} = v_n + h \cdot k_{1,v}, \quad \tilde{y}_{n+1} = y_n + h \cdot k_{1,y}$$
3. **Pendientes correctoras en el estado predicho:**
   $$k_{2,v} = 5.19 - 0.1 \cdot \tilde{v}_{n+1}, \quad k_{2,y} = \tilde{v}_{n+1}$$
4. **Paso Corrector (Promedio Trapezoidal final):**
   $$v_{n+1} = v_n + \frac{h}{2} \cdot (k_{1,v} + k_{2,v})$$
   $$y_{n+1} = y_n + \frac{h}{2} \cdot (k_{1,y} + k_{2,y})$$

---

### C. Método de Runge-Kutta 4 - RK4 (Cuarto Orden, $\mathcal{O}(h^4)$)

Utiliza un promedio ponderado de cuatro estimaciones de pendiente basadas en la Regla de Simpson:

1. **Etapa 1 (Inicio $t_n$):**
   $$k_{1,v} = 5.19 - 0.1 \cdot v_n, \quad k_{1,y} = v_n$$
2. **Etapa 2 (Punto medio con $k_1$, $t_n + h/2$):**
   $$v^{(1)} = v_n + \frac{h}{2} k_{1,v}, \quad k_{2,v} = 5.19 - 0.1 \cdot v^{(1)}, \quad k_{2,y} = v^{(1)}$$
3. **Etapa 3 (Punto medio refinado con $k_2$, $t_n + h/2$):**
   $$v^{(2)} = v_n + \frac{h}{2} k_{2,v}, \quad k_{3,v} = 5.19 - 0.1 \cdot v^{(2)}, \quad k_{3,y} = v^{(2)}$$
4. **Etapa 4 (Extremo final con $k_3$, $t_n + h$):**
   $$v^{(3)} = v_n + h \cdot k_{3,v}, \quad k_{4,v} = 5.19 - 0.1 \cdot v^{(3)}, \quad k_{4,y} = v^{(3)}$$
5. **Ponderación Final:**
   $$v_{n+1} = v_n + \frac{h}{6} \cdot (k_{1,v} + 2 k_{2,v} + 2 k_{3,v} + k_{4,v})$$
   $$y_{n+1} = y_n + \frac{h}{6} \cdot (k_{1,y} + 2 k_{2,y} + 2 k_{3,y} + k_{4,y})$$

---

## 6. Resultados Comparativos e Hitos de Evaluación

Resultados obtenidos con el paso estándar $h = 2\text{ s}$ en los hitos solicitados:

| Tiempo $t$ | $v_{\text{exacta}}$ (m/s) | $v_{\text{Euler}}$ (m/s) | Error $v$ Euler | $y_{\text{exacta}}$ (m) | $y_{\text{Euler}}$ (m) | Error $y$ Euler |
| :--------: | :-----------------------: | :----------------------: | :-------------: | :---------------------: | :--------------------: | :-------------: |
|  **2 s**   |         $9.4079$          |        $10.3800$         |    $0.9721$     |        $9.7213$         |        $0.0000$        |    $9.7213$     |
|  **4 s**   |         $17.1104$         |        $18.6840$         |    $1.5736$     |        $36.4961$        |       $20.7600$        |    $15.7361$    |
|  **10 s**  |         $32.8071$         |        $34.8934$         |    $2.0864$     |       $190.9294$        |       $170.0659$       |    $20.8635$    |
|  **20 s**  |         $44.8761$         |        $46.3273$         |    $1.4512$     |       $589.2390$        |       $574.7272$       |    $14.5118$    |

Comparación a $t = 20\text{ s}$ entre los tres métodos con $h = 2\text{ s}$:

| Método              | $v(20)$ (m/s) |  Error $v$ (m/s)   |  $y(20)$ (m)   |   Error $y$ (m)    | Evaluaciones $f$/paso |
| :------------------ | :-----------: | :----------------: | :------------: | :----------------: | :-------------------: |
| **Euler**           |   $46.3273$   |      $1.4512$      |   $574.7272$   |     $14.5118$      |          $1$          |
| **Heun**            |   $44.7664$   |      $0.1097$      |   $590.3355$   |      $1.0965$      |          $2$          |
| **RK4**             |   $44.8759$   | $\mathbf{0.00022}$ |   $589.2412$   | $\mathbf{0.00221}$ |          $4$          |
| **Solución Exacta** | **$44.8761$** |    **$0.0000$**    | **$589.2390$** |    **$0.0000$**    |           —           |

---

## 7. Estudio de Convergencia y Reducción del Error

Al dividir el paso $h$ sucesivamente entre 2 ($h \to h/2$), el error global acumulado se reduce a razón de $2^p$, donde $p$ es el orden del método:
$$\text{Razón} = \frac{\text{Error}(2h)}{\text{Error}(h)} \approx 2^p$$

| Paso $h$ (s) | Pasos $N$ | Error $y$ Euler | Razón Euler ($\mathcal{O}(h)$) | Error $y$ Heun | Razón Heun ($\mathcal{O}(h^2)$) | Error $y$ RK4 | Razón RK4 ($\mathcal{O}(h^4)$) |
| :----------: | :-------: | :-------------: | :----------------------------: | :------------: | :-----------------------------: | :-----------: | :----------------------------: |
|  **2.000**   |    10     |    $14.5118$    |               —                |    $1.0965$    |                —                |  $0.002214$   |               —                |
|  **1.000**   |    20     |    $7.1407$     |     $\times \mathbf{2.03}$     |    $0.2528$    |     $\times \mathbf{4.34}$      |  $0.000127$   |     $\times \mathbf{17.4}$     |
|  **0.500**   |    40     |    $3.5412$     |     $\times \mathbf{2.02}$     |    $0.0608$    |     $\times \mathbf{4.16}$      |  $0.000008$   |     $\times \mathbf{16.7}$     |
|  **0.250**   |    80     |    $1.7633$     |     $\times \mathbf{2.01}$     |    $0.0149$    |     $\times \mathbf{4.08}$      |  $0.000000$   |     $\times \mathbf{16.3}$     |
|  **0.125**   |    160    |    $0.8798$     |     $\times \mathbf{2.00}$     |    $0.0037$    |     $\times \mathbf{4.04}$      |  $0.000000$   |     $\times \mathbf{16.2}$     |

### Justificación de Eficiencia Computacional:

Para lograr un error milimétrico ($< 0.003\text{ m}$), Euler requeriría un paso de $h \approx 0.0003\text{ s}$ (**más de 66\,000 evaluaciones de función**). En cambio, RK4 lo consigue en apenas 10 pasos con 4 evaluaciones por paso (**40 evaluaciones totales**), demostrando por qué RK4 es el estándar en dinámica orbital y navegación aeroespacial.

---

## 8. Respuestas a las Preguntas de Reflexión

1. **¿Por qué era crucial verificar manualmente los cálculos antes del vuelo de John Glenn en 1962?**
   Las computadoras IBM 7090 eran novedosas y propensas a errores de compilación, redondeo o condiciones de contorno no contempladas. Un error ínfimo en el ángulo de entrada al corredor de reentrada atmosférica (ventana estrecha de ~1.5°) provocaría el rebote de la nave o su desintegración térmica. La verificación independiente de Katherine Johnson garantizó la seguridad de vidas humanas.
2. **¿Qué riesgos existirían al utilizar un paso $h$ demasiado grande?**
   Además del error de truncamiento inaceptable, para una EDO lineal $v' = a - bv$, Euler es asintóticamente estable solo si $h < \frac{2}{b}$. Con $b = 0.1\text{ s}^{-1}$, si $h > 20\text{ s}$ el método oscila y diverge hacia infinito numérico aunque la nave física vuele con suavidad.
3. **¿Por qué una aproximación puede parecer razonable y aun así acumular errores importantes?**
   El error local en cada paso es de fracciones de milímetros, pero se propaga y acumula sistemáticamente. Además, al integrar la altura ($y_{n+1} = y_n + h \cdot v_n$), el error de velocidad se integra doblemente en la posición. Visualmente, $574.7\text{ m}$ y $589.2\text{ m}$ lucen cercanos en una gráfica de 600 metros, pero $14.5\text{ metros}$ de discrepancia comprometen el momento exacto de eyección o apertura del paracaídas.
4. **¿Qué variables reales del lanzamiento no fueron consideradas?**
   Masa variable (consumo continuo de propelente, $\frac{dm}{dt} < 0$), empuje variable según altitud, resistencia cuadrática dependiente de la densidad atmosférica $\rho(y)$ y número de Mach, gravedad variable $g(y) = G M / (R + y)^2$ y la maniobra de cabeceo por gravedad (_gravity turn_).
5. **¿Por qué una trayectoria lunar requiere un modelo mucho más complejo?**
   Constituye un problema tridimensional de 3 cuerpos restringido (Tierra - Luna - Nave) que **no posee solución analítica cerrada**. Se compone de 6 a 12 EDOs no lineales acopladas, requiere métodos adaptativos (RK45 / Dormand-Prince) y métodos de disparo (_shooting methods_) para resolver el problema de valores de contorno en un vuelo de varios días.
