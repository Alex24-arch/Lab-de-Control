# Guía de los casos de prueba

Documento de apoyo para entender **qué se está simulando** y **por qué cada caso está incluido**.

---

## 1. De dónde sale el modelo

Un motor de CD controlado por armadura, cuando se desprecia la inductancia `La`, se comporta como un sistema de **primer orden**. Su función de transferencia (velocidad angular de salida sobre voltaje de armadura de entrada) es:

    G(s) = KM / (tau*s + 1)

Los dos coeficientes se obtienen de los cinco parámetros físicos:

    KM  = Kt / (Ra*b + Kt*Kb)
    tau = Ra*J / (Ra*b + Kt*Kb)

Observe que **ambos comparten el mismo denominador**:

    den = Ra*b + Kt*Kb

Ese término es el **amortiguamiento total** del motor y tiene dos contribuciones con significado físico distinto:

| Término | Origen | Qué representa |
|---|---|---|
| `Ra*b` | mecánico | la fricción del eje, que frena al motor |
| `Kt*Kb` | eléctrico | el frenado que produce la fuerza contraelectromotriz: al girar, el motor genera un voltaje que se opone al aplicado |

Si `den` fuera cero, el motor no tendría nada que lo frenara: giraría acelerando indefinidamente y el modelo dejaría de ser de primer orden (se convertiría en un integrador puro, `G(s) = K/s`). Por eso el script rechaza el caso `b = 0` **y** `Kb = 0` simultáneamente.

---

## 2. Qué significa cada coeficiente

**`KM` — ganancia de CD.** Es el valor al que tiende la salida cuando la entrada es un escalón unitario. Físicamente: cuántos rad/s alcanza el motor por cada voltio aplicado, una vez estabilizado.

**`tau` — constante de tiempo.** Mide qué tan rápido responde. Note que `tau` es proporcional a `J`: un motor con más inercia (o con una carga más pesada acoplada) tarda más en acelerar. Y es inversamente proporcional a `den`: más amortiguamiento, respuesta más rápida pero también más frenada.

---

## 3. De dónde salen los valores del gráfico

La respuesta al escalón unitario de un sistema de primer orden es:

    y(t) = KM * (1 - exp(-t/tau))

Todo lo que se dibuja sale de esa única expresión:

| Marca en el gráfico | Fórmula | Por qué |
|---|---|---|
| Valor final esperado | `y(inf) = KM` | el exponencial se anula cuando `t` crece |
| Marca en `t = 5*tau` | `y = 0.993*KM` | criterio práctico: a 5 constantes de tiempo ya se llegó al 99.3 %, se considera régimen permanente |
| `y(tau)` | `0.632*KM` | es la **definición** de constante de tiempo: el tiempo en que se alcanza el 63.2 % del valor final |
| Tiempo de asentamiento 2 % | `ts = 4*tau` | se exige `exp(-ts/tau) = 0.02`, o sea `ts = tau*ln(50) ≈ 3.912*tau`, que por convención se redondea a `4*tau` |
| Error de estado estacionario | `ess = 1 - KM` | la referencia vale 1 (escalón **unitario**); el error es lo que falta para alcanzarla |

**Punto clave sobre `ess`:** en este proyecto la planta está **en lazo abierto**, sin controlador. El error de estado estacionario existe simplemente porque `KM ≠ 1`. Corregirlo es precisamente el trabajo del regulador que se diseña más adelante en el curso.

---

## 4. Los cinco casos de prueba

Cada uno ejercita una situación distinta del sistema y una rama distinta del código.

### Caso A — Motor didáctico

    Kt = 0.01, Ra = 1.0, b = 0.1, Kb = 0.01, J = 0.01

| | |
|---|---|
| `den` | 0.1001 |
| `KM` | 0.0999001 |
| `tau` | 0.0999001 s |
| `ts` (2 %) | 0.3996 s |
| `ess` | 0.9001 |

Es el ejemplo clásico que aparece en la mayoría de los tutoriales de control. Aquí la fricción mecánica (`Ra*b = 0.1`) domina completamente sobre el frenado eléctrico (`Kt*Kb = 0.0001`, mil veces menor).

**Qué demuestra:** `KM` es muy pequeña, así que la salida se estabiliza en 0.1 mientras la referencia vale 1. El error de estado estacionario es del 90 % y domina visualmente todo el gráfico. Es el caso donde la flecha de `ess` se ve más grande.

### Caso B — Error pequeño pero visible

    Kt = 0.05, Ra = 2.5, b = 0.02, Kb = 0.05, J = 0.02

| | |
|---|---|
| `den` | 0.0525 |
| `KM` | 0.952381 |
| `tau` | 0.952381 s |
| `ts` (2 %) | 3.80952 s |
| `ess` | 0.047619 |

**Qué demuestra:** el error es de apenas 4.8 %, es decir, **justo por encima** del umbral del 2 % que usa el código para decidir si dibuja la flecha de `ess` o no. Es el caso límite del requisito "el error de estado estacionario (cuando sea visible)" del enunciado: sirve para verificar que la condición está bien puesta.

### Caso C — Seguimiento perfecto

    Kt = 0.1, Ra = 1.0, b = 0.09, Kb = 0.1, J = 0.005

| | |
|---|---|
| `den` | 0.1 |
| `KM` | 1.0 exacto |
| `tau` | 0.05 s |
| `ts` (2 %) | 0.2 s |
| `ess` | 0 |

Los números están escogidos para que `den = Ra*b + Kt*Kb = 0.09 + 0.01 = 0.1`, que es exactamente `Kt/1`, dando `KM = 1`.

**Qué demuestra:** la salida alcanza la referencia sin error. Ejercita la **rama contraria** del código: aquí `ess = 0` no supera el umbral, así que en vez de la flecha aparece el texto indicando que no es visible en la escala. Además `tau` es corto, por lo que se aprecia bien un asentamiento rápido.

### Caso D — Motor real (Maxon RE-25)

    Kt = 0.0234, Ra = 2.32, b = 1e-6, Kb = 0.0234, J = 1.07e-5

| | |
|---|---|
| `den` | 0.00054988 |
| `KM` | 42.5547 |
| `tau` | 0.0451444 s |
| `ts` (2 %) | 0.180578 s |
| `ess` | −41.5547 |

Estos son parámetros de hoja de datos de un motor comercial real. Note dos cosas físicamente típicas: `Kt` y `Kb` son numéricamente casi idénticas (en unidades SI deben serlo, es una consecuencia de la conservación de energía), y `b` es diminuta porque los rodamientos modernos tienen muy poca fricción — aquí el frenado es casi todo eléctrico.

**Qué demuestra:** `KM ≈ 42.5` rad/s por voltio, que es el orden de magnitud realista. Como la salida llega a 42 y la referencia unitaria vale 1, el escalón de entrada queda aplastado contra el eje horizontal. **Esto no es un error del programa:** confirma que los límites de los ejes se ajustan automáticamente al valor final, sea cual sea su magnitud. El `ess` sale negativo (−41.55) porque la salida se pasa muy por encima de la referencia.

### Caso E — Caso demostrativo

    Kt = 0.6, Ra = 1.0, b = 0.4, Kb = 0.5, J = 0.35

| | |
|---|---|
| `den` | 0.7 |
| `KM` | 0.857143 |
| `tau` | 0.5 s exacto |
| `ts` (2 %) | 2.0 s exacto |
| `ess` | 0.142857 |

Los parámetros están elegidos deliberadamente para que el gráfico quede legible: `tau = 0.5 s` y `ts = 2 s` son números redondos, la ventana de simulación es de 3 s, y `KM = 0.857` deja la curva ocupando casi toda la altura útil sin salirse.

**Qué demuestra:** es el caso donde **todas las anotaciones se ven bien separadas** a la vez: la curva llena el gráfico, la marca de `t = τ` no choca con la de `ts`, la banda del ±2 % es distinguible de la línea de valor final, y el `ess` del 14 % es lo bastante grande para dibujarse pero no tanto como para aplastar la curva. Es el que conviene usar para la captura de pantalla del informe.

---

## 5. Tabla comparativa

| Caso | `KM` | `tau` [s] | `ts` [s] | `ess` | Rama que ejercita |
|---|---|---|---|---|---|
| A | 0.0999 | 0.0999 | 0.3996 | 0.9001 | `ess` grande, domina el gráfico |
| B | 0.9524 | 0.9524 | 3.8095 | 0.0476 | `ess` en el límite del umbral |
| C | 1.0000 | 0.0500 | 0.2000 | 0.0000 | `ess = 0`, rama del texto alternativo |
| D | 42.5547 | 0.0451 | 0.1806 | −41.5547 | `KM >> 1`, autoescalado de ejes |
| E | 0.8571 | 0.5000 | 2.0000 | 0.1429 | todas las marcas bien separadas |

