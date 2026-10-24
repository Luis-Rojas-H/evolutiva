# Guia de estudio - Examen Parcial CC521
## Computacion Evolutiva (Prof. LANH)

**Alumno de referencia:** Luis Antonio Rojas Huaroc (20171581B)

Esta guia resume el material que el profesor dejo (semanas, labs, problemas desarrollados, script GA y la 2da calificada). Esta pensada para entender el "por que", no solo memorizar formulas.

---

## 0. Mapa de materiales (que estudiar y para que)

| Material | Donde esta | Que entra |
|---|---|---|
| Semana 02 Monte Carlo | `Semana2/Semana 02_Monte Carlo_.txt` + Ross | Estimacion por simulacion, integrales, convergencia |
| Transformada Inversa | `Semana3/metTI__discreto.txt` + Problemas | Generar VA discretas con F acumulada |
| Aceptacion-Rechazo | `Semana3/metA-R_discreto.txt` + Problemas | Generar con propuesta Q y constante c |
| Problemas desarrollados | `ProblemasDesarrollados_.pdf` / Semana5 | Monte Carlo, TI binomial negativa, A-R condicional, mezclas |
| 2do Lab dirigido | `2doLabDirigido_CS521_.pdf` | Seleccion (ruleta), enteros aleatorios, cruzamiento, mutacion |
| Script GA basico | `Semana5/script_GA basico.txt` | AG completo de clase (OBLIGATORIO usarlo si piden programa) |
| 2da Calificada | Semana7 | Esquemas, codificacion punto fijo, maximizar con AG |
| Goldberg | `Semana7/Goldberg_...pdf` | Teoria de esquemas / ideas del AG (guia, no citar si no piden) |

**Regla de oro del curso:** si piden "programa usado en clase", copia la estructura del script (select / crossover / mutate / translateDNA / get_fitness), no inventes otro AG.

---

## 1. Monte Carlo (Semana 2 + Ross)

### Idea central
Si puedes simular muchas veces un experimento aleatorio, el promedio de los resultados se acerca al valor esperado (ley de los grandes numeros).

### Patron de codigo del profesor
1. Generar muchas muestras.
2. Acumular con `cumsum / n`.
3. Graficar la convergencia contra el valor teorico.

### Ejemplo A: probabilidad de cara (moneda justa)
- Generas `n` ensayos: `1` si sale cara, `0` si no.
- Estimador: promedio acumulado.
- Valor teorico: `0.5`.

```python
n = 100
y = [int(rd.random() > 0.5) for i in range(n)]
yy = np.cumsum(y) / range(n)   # estimado acumulado
# yy[-1] ~ 0.5
```

### Ejemplo B: integral de exp(x) de 0 a 1
- Si U ~ Uniforme(0,1), entonces E[exp(U)] = integral_0^1 exp(x) dx = e - 1.
- Simulas `exp(U)` muchas veces y promedias.

```python
n = 250
y = [np.exp(rd.random()) for i in range(n)]
yy = np.cumsum(y) / range(1, n+1)
# yy[-1] ~ exp(1) - 1
```

### Ejemplo C (Problemas): E[N] con un while aleatorio
Patron tipico del profe:

```python
u = np.random.rand()
while u >= np.exp(-3):
    u = u * np.random.rand()
    N[i] = N[i] + 1
```

- Corres `k` replicas.
- Estimas `E[N]` con el promedio.
- Para `P[N = i]` cuentas frecuencias (`tolist().count`).

### Que te pueden preguntar
- Estimar una probabilidad o una esperanza.
- Graficar convergencia.
- Comparar estimado vs teorico.
- Interpretar: "con mas muestras el estimado se estabiliza".

### Errores frecuentes
- Olvidar dividir por n (usar suma cruda).
- Empezar el eje en 0 y dividir mal (`range(1,n+1)`).
- No dejar semilla / no reportar el ultimo valor estimado.

---

## 2. Generacion de variables aleatorias

### 2.1 Transformada inversa (discreta) - Semana 3

**Idea:** generas U ~ U(0,1) y buscas el menor j tal que F(j) >= U, donde F es la acumulada.

Pasos:
1. Tener probabilidades P = [p1, p2, ..., pm] (suman 1).
2. Calcular acumulada `P_cumsum`.
3. Para cada muestra: sacar u; avanzar j mientras `u > P_cumsum[j]`.
4. El valor generado es j+1 (si indexas desde 0).

```python
P = np.array([0.30, 0.20, 0.35, 0.15])
P_cumsum = np.cumsum(P)
u = np.random.rand()
j = 0
while u > P_cumsum[j]:
    j = j + 1
# valor = j + 1
```

**Uniforme discreta 1..n** (atajo del profe):

```python
x = np.floor(n * np.random.rand()) + 1
```

**Binomial negativa (Problemas desarrollados):**
- Empiezas en j = r con p_ = pr^r, F = p_.
- Mientras u >= F actualizas p_ con la recurrencia y sumas a F.
- Ese es el estilo "a mano" que le gusta al profesor (sin usar libreria de la distribucion).

### 2.2 Aceptacion-Rechazo (discreta) - Semana 3

**Idea:** quieres generar de P, pero es mas facil generar de una propuesta Q.
Eliges c >= max(P[i]/Q[i]).

Algoritmo:
1. Generar Y ~ Q.
2. Generar U ~ U(0,1).
3. Si U <= P(Y)/(c Q(Y)) aceptas Y; si no, repites.

```python
P = np.array([...])          # objetivo
Q = 0.10 * np.ones(n)        # propuesta (aqui uniforme)
c = np.max(P / Q)
y = floor(n*rand) + 1
u = rand
while u > P[y-1] / (c * Q[y-1]):
    y = floor(n*rand) + 1
    u = rand
# aceptar y
```

**Caso del PDF (condicional):**
Si quieres Y con la ley de X condicionada a {X >= k}:
- Propones X ~ Binomial(n,p).
- Aceptas solo si X >= k (equivalente a un A-R con indicador).

### 2.3 Mezcla de distribuciones (Problemas, item 4)
Si F = sum p_i F_i:
1. Eliges el componente i con probabilidad p_i (ruleta / TI).
2. Generas una muestra de ese F_i.

Ejemplo del profe:
- Con prob 1/3: Exp(2) usando `-0.5 * log(U)`
- Con prob 2/3: U(0,1)

### Que te pueden preguntar
- Implementar TI o A-R para un vector P dado.
- Comparar histograma simulado vs P teorica.
- Explicar por que aparece la constante c.
- Armar una mezcla a partir de una F partida a trozos.

---

## 3. Algoritmo genetico simple (Lab + Semana 5)

### 3.1 Piezas basicas (2do Lab dirigido)

#### A) Seleccion por ruleta (misma idea que TI)
Dadas probabilidades de seleccion de cada cromosoma:
- Acumulada + U aleatorio => eliges un indice.
- En el lab: simular secuencias de seleccion.

Nota: puede haber probabilidad 0 (ese individuo casi nunca se elige).

#### B) Entero uniforme entre a y b
Idea: mapear U(0,1) a enteros.
Comparar frecuencias simuladas vs esperadas (casi 1/(b-a+1) cada uno).

#### C) Cruzamiento de un punto
Padres P1, P2 y punto c:
- Hijo1 = P1[0:c] + P2[c:]
- Hijo2 = P2[0:c] + P1[c:]

Codigo estilo lab:

```python
def crossover_(parent_1, parent_2, cross_point):
    parent_1_aux = parent_1.copy()
    parent_1 = np.concatenate((parent_1[0:cross_point], parent_2[cross_point:DNA_SIZE]))
    parent_2 = np.concatenate((parent_2[0:cross_point], parent_1_aux[cross_point:DNA_SIZE]))
    return cross_point, parent_1, parent_2
```

#### D) Mutacion de un bit
Con probabilidad pm volteas el bit (0<->1).
Si haces 1000 intentos, el numero esperado de mutaciones es `1000 * pm`.
Compara esperado vs observado para pm = 0.1, 0.01, 0.001.

### 3.2 Script GA de clase (Semana 5) - MEMORIZAR ESTRUCTURA

Parametros tipicos del script:

- `DNA_SIZE` = longitud del cromosoma (l)
- `POP_SIZE` = n (tamano de poblacion)
- `CROSS_RATE` = pc
- `MUTATION_RATE` = pm
- `N_GENERATIONS`
- `X_BOUND` = [xmin, xmax]

Funciones clave:

1. `F(x)`: funcion objetivo (maximizar).
2. `get_fitness(pred)`: hace el fitness no negativo:
   `pred + 1e-3 - min(pred)`
3. `translateDNA(pop)`: binario -> real.
4. `select(pop, fitness)`: ruleta con `np.random.choice(..., p=fitness/sum)`.
5. `crossover(parent, pop)`: con prob pc mezcla bits con otro individuo (mascara aleatoria).
6. `mutate(child)`: bit a bit con pm.

Ciclo generacional (orden importante):

```text
evaluar F
fitness = get_fitness(...)
seleccion
para cada padre:
    cruzar
    mutar
    reemplazar padre por hijo
```

**Atencion (observacion del profesor en la calificada):**
Si piden resolver un maximo con AG y "usar el programa de clase", NO escribas un AG distinto.
Cambia solo:
- la funcion F,
- el dominio X_BOUND,
- DNA_SIZE segun la precision,
- y justifica n, pc, pm (puedes heredar los del script: n=50, pc=0.25, pm=0.01).

### 3.3 Como elegir parametros (lo que piden justificar)

| Parametro | Significado | Criterio practico en este curso |
|---|---|---|
| n (POP_SIZE) | tamano poblacion | 50 como en clase |
| precision | paso del mapeo real | define l: (b-a)/(2^l - 1) <= precision |
| ps | seleccion | proporcional al fitness (ruleta del script) |
| pc | cruzamiento | 0.25 en el script de clase |
| pm | mutacion | 0.01 en el script de clase |

Dominio: si la funcion crece sin limite (ej. 3x^10), DEBES acotar el intervalo; si no, no hay maximo finito. En la calificada se uso [0,1].

---

## 4. Esquemas y codificacion (eje de la 2da calificada)

### 4.1 Longitud del cromosoma
Si el espacio binario tiene N puntos:

```text
2^l = N  =>  l = log2(N)
```

Ejemplo calificada: N = 2097152 = 2^21 => l = 21.

### 4.2 Cantidades de esquemas (leer el enunciado con cuidado)

Definiciones utiles:

1. **Esquemas en UN individuo:** en cada posicion dejas el bit o pones `*`.
   - Respuesta: `2^l`

2. **Puntos de busqueda:**
   - Respuesta: `2^l` (a veces es el mismo dato del enunciado; no te enredes)

3. **Esquemas DISTINTOS en todo el espacio** (teoria clasica / Goldberg):
   - Cada posicion: `0`, `1` o `*`
   - Respuesta: `3^l`

4. **Total "sumando" esquemas de todos los puntos** (conteo con repeticion):
   - Cada punto aporta `2^l` y hay `2^l` puntos
   - Respuesta: `2^l * 2^l = 2^(2l)`

**Leccion de la calificada:** el profesor es muy literal.
En la 2da calificada marco incorrecto responder solo `3^l` en "numero TOTAL de esquemas".
La lectura que calzo con su correccion fue el conteo total `2^(2l)`.
En un parcial: si dice "distintos"/"diferentes" usa `3^l`; si dice "total" sin mas, piensa y justifica.

### 4.3 Cotas en una poblacion de tamano n
- Inferior: todos iguales => `2^l`
- Superior: `n * 2^l`

(A veces se menciona que no puede pasar de `3^l` esquemas distintos.)

### 4.4 Individuos de t a t+1 (trampa tipica)
Si preguntan:

> cota inferior del numero esperado de **individuos** a obtener de t a t+1

NO respondas `n^3` (eso es paralelismo implicito de **esquemas**).
Respuesta literal: **n** (la poblacion nueva sigue siendo de tamano n).

Ejemplo: n=50 => respuesta 50.

### 4.5 Codificacion de punto fijo (varias variables)

Para cada variable xi en [ai, bi] con precision ei:

```text
(bi - ai) / (2^{li} - 1)  <=  ei
=>  2^{li} - 1  >=  (bi - ai)/ei
=>  eliges el menor li que cumpla
```

Luego concatenas: `[bits_x | bits_y | bits_z]`.

Mapeo (decodificar):

```text
x = a + int(bits) / (2^l - 1) * (b - a)
```

Codificar (inverso, redondeando al entero mas cercano).

Ejemplo de la calificada:
- x en [-20, 125], prec 0.5 => lx=9
- y en [0, 1.2e6], prec 10000 => ly=7
- z en [-0.1, 1.0], prec 0.001 => lz=11
- Total bits = 27

Puntos extremos:
- limites inferiores => todo 0
- limites superiores => todo 1

---

## 5. Trampas / estilo del profesor (importante)

1. **Lee el sustantivo:** individuos vs esquemas vs puntos.
2. **Puede repetir un dato** (ej. pedir puntos de busqueda cuando ya dio N). Responde simple.
3. **Puede no dar la funcion** y solo pedir codificacion: no inventes un f.
4. **Puede no dar el dominio** en un maximo: debes asumirlo y justificarlo.
5. **Si pide programa de clase:** usa el script GA, no uno "mejor" o distinto.
6. **Notacion rara:** `1.2(106)` = `1.2e6`; un punto mal escrito tipo `(125,1.2e6.1.0)` suele ser `(125, 1.2e6, 1.0)`.
7. Redaccion clara, sin adornos; muestra numeros y una justificacion corta.

---

## 6. Formulas rapidas (cheatsheet)

```text
l desde N puntos binarios:     2^l = N
esquemas por individuo:        2^l
esquemas distintos (espacio):  3^l
total con multiplicidad:       2^(2l)
cotas en poblacion n:          [2^l , n*2^l]
individuos t->t+1:             n

bits por variable:             menor l con (2^l - 1) >= (b-a)/precision
decode:                        a + int/(2^l-1)*(b-a)

TI discreta:                   u vs acumulada F
A-R:                           aceptar si u <= P(y)/(c Q(y)), c>=max(P/Q)

Monte Carlo:                   promedio de muchas simulaciones -> E[.]
```

---

## 7. PARCIAL SIMULADO (estilo del profesor)

Tiempo sugerido de practica: 90-120 min. Hazlo primero sin mirar respuestas.

---

### Pregunta 1 — Monte Carlo (8 pts)

Se define el siguiente experimento (estilo problemas desarrollados):

```python
u = random()
N = 0
while u >= exp(-2):
    u = u * random()
    N = N + 1
```

a) Explica en 3-4 lineas que estima el promedio de N en k replicas.  
b) Escribe el esquema de programa para estimar E[N] y graficar la convergencia.  
c) Como estimarias P[N=0], P[N=1], P[N=2] por simulacion?

#### Respuesta

**a)** Cada replica produce un entero N >= 0. El promedio de muchas replicas estima E[N] por Monte Carlo (ley de grandes numeros).

**b)** Patron:

```python
k = 1000
N = np.zeros(k)
for i in range(k):
    u = np.random.rand()
    while u >= np.exp(-2):
        u = u * np.random.rand()
        N[i] = N[i] + 1
N_acum = np.cumsum(N) / np.arange(1, k+1)
plt.plot(N_acum)
print(N_acum[-1])  # estimado de E[N]
```

**c)** Cuentas frecuencias:

```python
for i in [0,1,2]:
    print(i, N.tolist().count(i) / k)
```

---

### Pregunta 2 — Transformada inversa y A-R (10 pts)

Sea P = (0.1, 0.25, 0.35, 0.30) sobre los valores {1,2,3,4}.

a) Genera UNA muestra con transformada inversa. Muestra la acumulada y el criterio con u.  
b) Propón Q uniforme discreta en {1,2,3,4} y calcula c para A-R.  
c) Escribe el loop de aceptacion-rechazo (estilo Semana 3).

#### Respuesta

**a)**  
F = (0.10, 0.35, 0.70, 1.00).  
Si por ejemplo u=0.42: como 0.35 < 0.42 <= 0.70 => sale 3.

**b)** Q(i)=1/4=0.25.  
P/Q = (0.40, 1.00, 1.40, 1.20) => c = 1.40.

**c)**

```python
P = np.array([0.1, 0.25, 0.35, 0.30])
Q = 0.25 * np.ones(4)
c = np.max(P/Q)
y = int(np.floor(4*np.random.rand())) + 1
u = np.random.rand()
while u > P[y-1]/(c*Q[y-1]):
    y = int(np.floor(4*np.random.rand())) + 1
    u = np.random.rand()
# y aceptado
```

---

### Pregunta 3 — Lab AG: seleccion, cruce, mutacion (10 pts)

a) Con P_sel = (0.2, 0.5, 0.3) para 3 cromosomas, simula 1 seleccion por ruleta (acumulada).  
b) Cruza en el punto 3:  
   A = 110100  
   B = 001111  
   Da los dos hijos.  
c) Si pm=0.01 y haces 1000 pruebas sobre UN bit, cual es el numero esperado de mutaciones? Que compararias en un programa?

#### Respuesta

**a)** F=(0.2, 0.7, 1.0). Si u=0.55 => cae en el segundo (0.2 < 0.55 <= 0.7) => cromosoma 2.

**b)** Punto 3:  
Hijo1 = 110 + 111 = 110111  
Hijo2 = 001 + 100 = 001100

**c)** Esperado = 1000 * 0.01 = 10.  
En el programa cuentas cuantas veces realmente muto y comparas contra 10 (y lo mismo para otros pm).

---

### Pregunta 4 — Esquemas (estilo calificada 2) (10 pts)

Un espacio de busqueda binario tiene 1048576 puntos. Poblacion n=40.

a) l = ?  
b) Numero de puntos de busqueda.  
c) Esquemas contenidos en cada individuo.  
d) Cotas inferior y superior de esquemas en la poblacion.  
e) Cota inferior del numero esperado de individuos a obtener de t a t+1.  
f) (extra de teoria) Si preguntaran esquemas DISTINTOS del espacio, cuanto seria?

#### Respuesta

a) 2^l = 1048576 = 2^20 => **l=20**  
b) **1048576**  
c) **2^20 = 1048576**  
d) Inferior **2^20**; superior **40 * 2^20 = 41943040**  
e) Piden individuos => **40** (NO 40^3)  
f) Distintos: **3^20**  
   Total con multiplicidad (lectura "total" de la calificada): **2^40**

---

### Pregunta 5 — Codificacion punto fijo (8 pts)

Maximizar/minimizar no importa: solo codificar.
- x en [0, 10], precision 0.01
- y en [-5, 5], precision 0.1

a) Bits minimos lx, ly y total.  
b) Cadena del punto (0, -5).  
c) Cadena del punto (10, 5).  
d) Entero y bits de x=2.5 (con el lx que hallaste).

#### Respuesta

**a)**  
x: (10-0)/0.01 = 1000 => 2^lx - 1 >= 1000 => lx=10 (1023>=1000)  
y: (5-(-5))/0.1 = 100 => 2^ly - 1 >= 100 => ly=7 (127>=100)  
Total = **17 bits**

**b)** limites inferiores => todo ceros: `0000000000` + `0000000`  
**c)** limites superiores => todo unos: `1111111111` + `1111111`  
**d)** ix = round( (2.5-0)/(10-0) * 1023 ) = round(255.75) = 256  
256 en 10 bits = `0100000000`

---

### Pregunta 6 — AG con script de clase (10 pts)

Hallar el maximo de f(x) = x * sin(10*pi*x) + 1 en el intervalo del script de clase X_BOUND=[-1, 2]  
(o, variante calificada: f(x)=3x^10 en [0,1]).

a) Justifica n, precision/l, ps, pc, pm.  
b) Indica que partes del script cambias y cuales NO.  
c) Que grafico entregarias para "convergencia".

#### Respuesta

**a)** Ejemplo justificado al estilo clase:
- n=50 (POP_SIZE del script)
- precision: eliges l=DNA_SIZE; en el script de clase viene 22 para [-1,2]
- ps: seleccion proporcional (`select`)
- pc=0.25 (`CROSS_RATE`)
- pm=0.01 (`MUTATION_RATE`)

Para 3x^10 en [0,1] con precision 0.001: l=10, mismo n/pc/pm.

**b)** Cambias F (y si hace falta X_BOUND / DNA_SIZE).  
NO cambias la logica de select/crossover/mutate/get_fitness ni el orden del loop.

**c)** Curva de mejor fitness (y opcional promedio) vs generacion.  
Ademas, como en clase, se puede mostrar la poblacion final sobre f(x).

---

## 8. Checklist dia del parcial

- [ ] Se importar `numpy` / `matplotlib` como en los scripts
- [ ] Monte Carlo: promedio acumulado + grafico
- [ ] TI: acumulada + while con u
- [ ] A-R: c = max(P/Q) + while de rechazo
- [ ] Ruleta de seleccion (lab)
- [ ] Cruzamiento de un punto (lab) y saber que el script de clase usa mascara de bits
- [ ] Mutacion bit a bit y valor esperado k*pm
- [ ] Esquemas: 2^l, 3^l, 2^(2l), cotas n, y NO confundir con n^3 si piden individuos
- [ ] Bits por precision: menor l con (2^l-1) >= rango/prec
- [ ] Si piden AG: partir del `script_GA basico.txt`

---

## 9. Mini glosario

- **Cromosoma / DNA:** cadena binaria de longitud l.
- **Poblacion:** conjunto de n cromosomas.
- **Fitness:** valor usado para seleccionar (a veces desplazado para que sea > 0).
- **Esquema:** patron con 0, 1 y `*` (no importa).
- **pc / pm:** probabilidad de cruzamiento / mutacion.
- **Punto fijo:** discretizar un real a un entero de l bits y mapear al intervalo [a,b].

---

## 10. Referencia rapida a la 2da calificada (resuelta)

1. l=21  
   - a) total esquemas (lectura calificada): 2^42  
   - b) puntos: 2^21  
   - c) por individuo: 2^21  
   - d) cotas: 2^21 y 50*2^21  
   - e) individuos t->t+1: 50  

2. bits 9+7+11=27; cadenas extremo todo0/todo1; (50,1e5,0.597) -> `011110111000101110100010001`

3. AG del script de clase con F(x)=3x^10, dominio [0,1], l=10, n=50, pc=0.25, pm=0.01 + grafico de convergencia.

Notebook: `Semana7/2da_Calificada_CS521_Luis_Rojas.ipynb`

---

Fin de la guia. Estudia primero los `.txt` de Semana2/3/5 (son el estilo exacto del profesor) y luego practica el parcial simulado escribiendo codigo a mano / en Colab.
