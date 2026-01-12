# Max-Cut + Ising + Simulated Annealing + 1 parámetro

Este proyecto tiene como objetivo:

> **Explorar cómo la temperatura inicial afecta la calidad de las soluciones al resolver Max-Cut mediante un solver estocástico tipo Ising.**

## Estructura general

```mermaid
    graph LR
    A[Problema]
    B[Modelo Físico]
    C[Dinámica]
    D[Diseño experimental]
    E[Resultados]
    A --> B
    B --> C
    C --> D
    D --> E

    style A fill: #3f2ea0ff
    style B fill: #6352c7ff
    style C fill: #5b5099ff
    style D fill: #7067a4ff
    style E fill: #9a8edaff
```
**Pregunta foco**
>_¿Cómo influye la temperatura inicial en la capacidad del sistema para encontrar buenas soluciones?_
## Problema 
En esta sección se busca construir y entender el problema del proyecto, para lo cual se buscará responder la pregunta _¿Qué problema se está estudiando realmente?_
Para este proyecto se estudiará **Max-Cut**, el cual es un problema de optimización combinatorio NP-Hard, sin embargo conceptualmente ese no es el foco principal, se escogió porque se trata de un ejemplo mínimo de un problema NP-Hard que puede formularse como un paisaje de energía, algo así como mi "Hello, World!" o mi "Oscilador Armónico" de este campo. 

Intuitivamente este problema lo podemos entender teniendo un grafo (nodos conectados por aristas), donde cada una de sus aristas tiene un peso (qué tan importante es esta), la tarea que se tiene es que se quiere dividir a este grafo en dos grupos de tal forma que la suma de los pesos de las aristas que cruzan entre los grupos sea máxima. Las reglas son que las aristas que conectan nodos del mismo grupo no cuentan y las que cruzan de un grupo a otro si. 

_Ejemplo de un grafo y una de sus posibles configuraciones_
```mermaid 
    graph LR
    A[a] --- B[b]
    A --- C[c]
    B --- D[d]
    C --- E[e]
    D --- F[f]
    E --- F
    B --- E
    C --- D

    G[Grupo A] <--> H[Grupo B]
    H <--> I
    I[N - Nodos] <--> J[2ᴺ Particiones posibles]

    style A fill: #632a7eff
    style B fill: #2d5b97ff
    style C fill: #2d5b97ff
    style D fill: #632a7eff
    style E fill: #2d5b97ff
    style F fill: #2d5b97ff
    style G fill: #632a7eff
    style H fill: #2d5b97ff
    style I fill: #899950ff
    style J fill: #899950ff

```
Otro punto importante es mencionar el enfoque que se le dará, en este caso en vez de pensar en dividir nodos, se visualiza como **la asignación de estados binarios a un sistema con interacciones**, en donde cada nodo es una variable binaria que recibe su estado, mientras que el sistema completo tiene una configuración y una energía asociada. Por lo que se buscará encontrar configuraciones de baja energía. 
Además no se busca un óptimo exacto, sino entender el comportamiento del solver, en general estudiarlo como un sistema físico explorando su espacio de estados. 

## Modelo Físico 
Definimos cada nodo como una varible física binaria (spin)

$$ s_i \in \{-1, 1\} $$

y una configuración del sistema como 

$$ \textbf{s} = (s_1, s_2 , ... , s_N)$$

por lo que una solución al problema es un estado físico del sistema.
Teniendo que la energía del sistema estará dada por:

$$ E(\textbf{s}) = \sum_{(i,j) \in E} J_{ij}s_is_j$$

por lo que:

- Si dos spines contectado son iguales contribuyen positivamente
- Si dos spines conectados son opuestos contribuyen negativamente 
- $J_{ii} < 0$ por lo que el sistema prefiere spines conectados de manera opuesta 
- Mínimizar la energía del sistema será equivalente a maximizar el corte. 

Cabe mencionar que se está usando un modelo de Ising sin campos externos y sin ninguna dependencia temporal en el Hamiltoniano. 

> **Nota importante:** Una vez formulado el problema como un sistema físico, la pregunta deja de ser “cómo encontrar la mejor partición” y se convierte en “cómo permitir que el sistema explore eficientemente su paisaje de energía”.

## Dinámica 
En esta sección se busca definir una regla de evolución que le diga al sistema cómo moverse en el paisaje de energía. En este caso **Simulated Annealing** está inspirado en un proceso físico reconocido de enfriar lentamente un material permitiendo que explore estados y se "congele" en un mínimo profundo. De manera general un paso en la dinámica se compone de:

```mermaid
    graph TD
    A[Se tiene una configuración **s**]
    B[Se propone cambiarla ligeramente]
    C[Se evalúa si el cambio es aceptado]

    A --> B
    B --> C

    style A fill: #77375dff
    style B fill: #77375dff
    style C fill: #77375dff
```

Para este caso la **regla de aceptación** la define el **algoritmo de Metropolis**. 

Se tiene un spin $s_i$, se propone $s_i → -s_i$, además se tiene el cambio de energía dado por:

$$\Delta E = E_{new} - E_{old} $$

- Si $\Delta E < 0$ entonces se acepta 
- Si $\Delta E > 0$ entonces se acepta con la probabilidad
    
    $$P = \exp(-\Delta E / T)$$

en este caso el rol de la temperatura controla qué tan probable es aceptar movimientos "malos".
Analizando los casos límites

- $T → 0$: Solo se aceptan mejoras 
- $T → ∞$: Se aceptan casi todo, equivalente a un random walk

Además, para el enfriamiento se usará un Schedule lineal(la temperatura disminuye linealmente) ya que se está priorizando la claridad y este resulta suficiente 

$$ T(t) = T_0 - (T_0 - T_f)\dfrac{t}{N_{steps}}$$

es importante recordar que para este experimento se van a fijar el resto de los parámetros excepto justamente la $T_0$.

De esta manera podemos obtener las siguientes ideas principales:

- La dinámica estocástica permite explorar el paisaje de energía.
- La regla de Metropolis introduce aceptación probabilística.
- La temperatura controla el balance exploración–explotación.
- El enfriamiento progresivo favorece la convergencia.
- El comportamiento final depende críticamente de los parámetros.
- El solver es inherentemente probabilístico.

## Diseño experimental
En esta sección se busca responder a la pregunta base _¿Cómo cambia el comportamiento de un solver estocástico cuando se varía un parámetro físico?_ Por lo que vale la pena definir cuales serán los parámetros fijos en el experimento.

**Parámetros fijos**
- Grafo 
- Número de Nodos $N$
- Pesos de las aristas 
- Número de pasos del SA

**Parámetro variable**
- Temperatura inicial $T_0$

Para cada valor de $T_0$ 

```mermaid 
    graph LR
    A[1 <br> Inicializar el sistema con spins aleatorios]
    B[2 <br> Ejecutar Simulated Anneling]
    C[3 <br> Registrar: Energía final, mejor energía alcanzada, valor del corte]
    D[4 <br> Repetir M veces]
    
    A --> B
    B --> C
    C --> D

    style A fill: #7067a4ff
    style B fill: #7067a4ff
    style C fill: #7067a4ff
    style D fill: #7067a4ff
```
El esquema anterior se plasmó en la siguiente estructura de código dentro del notebook `main.ipynb`.

```mermaid
    graph TD
    A[1 <br> Crear un grafo aleatorio usando el modelo Erdős-Rényi] --> B[ 2 <br> Definición del valor del corte y de la energía Ising]
    B --> C[3 <br> Calculo del cambio de energía al voltear un spin]
    C --> D[4 <br> Implementación de Simulated Annealing]
    D --> E[5 <br> Definición del experimento, barrido en T_0 para múltiples corridas]
    
    style A fill: #915a7bff
    style B fill: #915a7bff
    style C fill: #915a7bff
    style D fill: #915a7bff 
    style E fill: #915a7bff
```