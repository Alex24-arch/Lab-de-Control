# Guía del Proyecto 3 — Diseño de compensadores

Documento de apoyo: qué se está diseñando y de dónde sale cada fórmula.

---

## 1. El cambio respecto a los proyectos anteriores

| Proyecto | Pregunta que responde |
|---|---|
| 1 | ¿Cómo se comporta esta planta? |
| 2 | ¿Es estable? ¿Para qué rango de K? |
| **3** | **¿Qué hay que agregarle para que se comporte como yo quiero?** |

Los dos primeros eran de **análisis**. Éste es de **diseño**: se parte de un comportamiento deseado y se calcula el controlador que lo consigue.

---

## 2. Los dos criterios del lugar de las raíces

Todo el proyecto descansa en una observación simple. Un punto `sd` del plano `s` es polo de lazo cerrado si cumple la ecuación característica:

    1 + Gol(sd) = 0        ⟹        Gol(sd) = −1

Ahora bien, `−1` es un número complejo con **magnitud 1** y **ángulo 180°**. Esa única condición compleja se separa en dos condiciones reales:

| Criterio | Condición | Qué determina |
|---|---|---|
| **Ángulo** | `∠Gol(sd) = ±180°` | La **forma** del lugar: por dónde pasa. No depende de la ganancia. |
| **Magnitud** | `|Gol(sd)| = 1` | **Qué ganancia** corresponde a cada punto del lugar. |

Esta separación es la clave del diseño. El criterio del ángulo no involucra a `K`, así que define la geometría del problema; el de la magnitud se aplica después, solo para poner el número.

---

## 3. El procedimiento de diseño

### Paso 1 — Elegir `sd`

El usuario decide dónde quiere los polos dominantes de lazo cerrado. Puede hacerlo con el ratón, escribiendo coordenadas, o a partir de especificaciones. Esta última es la más útil en la práctica: para un sistema de segundo orden estándar,

    sd = −ζ·ωₙ ± j·ωₙ·√(1−ζ²)

donde `ζ` (amortiguamiento) controla el sobreimpulso y `ωₙ` la velocidad:

    Mp = e^(−πζ/√(1−ζ²)) × 100 %          ts(2 %) ≈ 4/(ζ·ωₙ)

O sea: si le piden "sobreimpulso menor al 16 % y asentamiento en 2 s", eso se traduce en `ζ ≈ 0.5` y `ωₙ ≈ 4`, y de ahí sale `sd`.

### Paso 2 — Medir la deficiencia de ángulo

Se evalúa el ángulo que la planta aporta en `sd`:

    ∠G(sd) = Σ ∠(sd − zᵢ) − Σ ∠(sd − pᵢ)

Los ceros aportan ángulo **positivo**, los polos **negativo**. Si el resultado ya es −180°, `sd` está sobre el lugar de las raíces y basta ajustar la ganancia. Si no, la diferencia es lo que falta:

    φ = −180° − ∠G(sd)

**Ese φ es exactamente el ángulo que debe aportar el compensador.** De ahí la frase del enunciado: el compensador es el *complemento* de la planta.

| Signo de φ | Significado | Solución |
|---|---|---|
| φ > 0 | Falta ángulo | **Adelanto** o **PD** (un cero aporta ángulo positivo) |
| φ ≈ 0 | `sd` ya está en el lugar | Solo ganancia |
| φ < 0 | Sobra ángulo | **Atraso**, o reubicar `sd` |

### Paso 3 — Ubicar cero y polo del compensador

**PD (un solo cero):** el cero debe aportar todo el ángulo. Con `sd = σ + jω`, el ángulo del vector desde el cero `xz` hasta `sd` es `atan(ω/(σ−xz))`, así que:

    xz = σ − ω/tan(φ)

Funciona solo si `0 < φ < 180°`, porque es lo máximo que un cero puede aportar. El PD es simple pero amplifica ruido de alta frecuencia — por eso en la práctica se prefiere el adelanto.

**Adelanto (cero + polo), método de la bisectriz:** hay infinitas combinaciones de cero y polo que suman `φ`. La bisectriz elige la que **maximiza el adelanto de fase para una separación polo-cero dada**, o sea la más eficiente. La construcción:

1. Desde `sd` se trazan dos rayos: uno horizontal hacia la izquierda (180°) y otro hacia el origen.
2. Se bisecta el ángulo entre ellos.
3. Desde la bisectriz se abren dos rayos a `±φ/2`.
4. Donde cada rayo corta el eje real quedan el cero y el polo.

La intersección de un rayo que sale de `(σ, ω)` con ángulo `θ` contra el eje real es `x = σ − ω/tan(θ)`.

**Atraso:** no busca cambiar la forma del lugar sino subir la ganancia estática. Se colocan cero y polo **muy cerca del origen**, con relación `zc/pc = β` igual a la mejora deseada del error estacionario. Al estar tan cerca del origen y tan juntos entre sí, el ángulo neto que aportan en `sd` es casi nulo — el lugar apenas se mueve — pero la ganancia de CD se multiplica por `β`.

### Paso 4 — Fijar la ganancia

Ya con la geometría resuelta, el criterio de la magnitud da el número:

    |Gc(sd)·G(sd)| = 1        ⟹        Kc = 1 / |Gc₁(sd)·G(sd)|

donde `Gc₁` es el compensador con ganancia unitaria.

### Paso 5 — Verificar

Se arma la nueva ecuación característica y se comprueba que `sd` sea realmente una de sus raíces. El script lo hace y reporta el error; debe salir del orden de `10⁻¹⁵`.

---

## 4. Verificación a mano — Caso A

    G(s) = 4 / [s(s+2)]        sd deseado = −2 + j3.4641   (ζ = 0.5, ωₙ = 4)

### Deficiencia de ángulo

Vectores desde cada polo hasta `sd`:

- Desde el polo en `0`: `sd − 0 = −2 + j3.4641`, ángulo = `atan2(3.4641, −2)` = **120°**
- Desde el polo en `−2`: `sd + 2 = 0 + j3.4641`, ángulo = **90°**

Como son polos, aportan negativo:

    ∠G(sd) = −120° − 90° = −210°
    φ = −180° − (−210°) = **+30°**

Faltan 30 grados ⟹ compensador de adelanto.

### Bisectriz

- Rayo de `sd` al origen: `0 − sd = 2 − j3.4641`, ángulo = **−60°**
- Rayo horizontal izquierdo: **180°**
- Bisectriz: `(−60 + 180)/2 = **60°**`
- Rayo del cero: `60 + 15 = 75°` → `xz = −2 − 3.4641/tan(75°) = **−2.928**`
- Rayo del polo: `60 − 15 = 45°` → `xp = −2 − 3.4641/tan(45°) = **−5.464**`

### Ganancia

    Gc₁(sd) = (sd + 2.928)/(sd + 5.464) = (0.928 + j3.464)/(3.464 + j3.464)
    |Gc₁(sd)| = 3.586/4.899 = 0.7321
    |G(sd)| = 4/(|sd|·|sd+2|) = 4/(4 × 3.4641) = 0.2887
    Kc = 1/(0.7321 × 0.2887) = **4.732**

### Resultado

    Gc(s) = 4.732 (s + 2.928) / (s + 5.464)

Ecuación característica compensada: raíces en `−2 ± j3.4641` (exactamente `sd`) y `−3.4641`.

Este es el ejemplo clásico del Ogata, y el script debe reproducir estos mismos números.

---

## 5. Las cinco plantas de ejemplo

| # | `G(s)` | Particularidad |
|---|---|---|
| A | `4/[s(s+2)]` | Ejemplo canónico de libro de texto; ideal para validar |
| B | `1/[(s+1)(s+2)(s+3)]` | Tercer orden; el compensador debe vencer más ángulo negativo |
| C | `1/[s(s+1)(s+5)]` | Con integrador y polo lejano; margen estrecho |
| D | `1/[(s+0.5)(s+3)]` | Sin integrador ⟹ error estacionario apreciable; buen caso para atraso |
| E | `1/[s(s−1)]` | **Planta inestable.** El compensador la estabiliza |

El caso E es el más interesante conceptualmente: la planta tiene un polo en `+1`, y en el proyecto 2 se vio que **ningún valor de K** la estabilizaba con realimentación proporcional pura. Aquí, con un compensador de adelanto, sí se logra. Es la demostración de por qué a veces no basta con ajustar la ganancia: hay que agregar dinámica.

---

## 6. Advertencia sobre la dominancia

El método coloca **dos** polos donde se pidió, pero el sistema tiene más. Los polos deseados solo gobiernan la respuesta si los demás quedan **bastante más a la izquierda** — la regla práctica es un factor de 5 o más en la parte real.

Si no hay dominancia, la respuesta real no se parecerá a la del segundo orden ideal que se usó para elegir `sd`, y el sobreimpulso o el asentamiento saldrán distintos de lo especificado. El script calcula esa razón y lo advierte.

Por eso la opción 9 (comparar respuestas) es importante: **cierra el ciclo**. Se diseñó a partir de una especificación en el plano `s`, y ahí se verifica en el dominio del tiempo si efectivamente se cumplió.

---

## 7. Experimentos sugeridos

- **Caso A, opción 7 → 3 (desde especificaciones) con ζ=0.5, ωₙ=4.** Debe reproducir el compensador del libro: cero en −2.93, polo en −5.46, Kc = 4.73.
- **Caso E.** Compruebe que la planta inestable queda estable. Mire el root locus compensado: las ramas que se iban al semiplano derecho ahora se doblan hacia la izquierda.
- **Caso D con atraso.** El lugar de las raíces casi no cambia, pero el error estacionario baja según el factor `β` elegido. Contrástelo con la `Kp` que reporta la opción 8.
- **Pida un `sd` muy a la izquierda** (respuesta muy rápida). Verá que la `Kc` requerida se dispara: la velocidad se paga con esfuerzo de control. Es una limitación física real, no del método.
- **Opción 6, arrastre un polo hacia el semiplano derecho** y observe cómo el lugar de las raíces se deforma en vivo.
