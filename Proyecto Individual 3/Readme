# EL-5409 — Proyecto corto 3

Diseño interactivo de compensadores sobre el lugar de las raíces, en una consola de MATLAB/Octave con selección por ratón.

**Estudiante:**  Santos Alexander López Valles— **Carné:** 2023057479

## Descripción

A partir de los ceros y polos de la planta, el script construye `G(s)` en formato racional, extrae la ecuación característica y dibuja el lugar de las raíces. El usuario selecciona con el ratón dónde desea que queden los polos dominantes de lazo cerrado, y el programa calcula el **compensador `Gc(s)`** necesario para conseguirlo, aplicando el criterio del ángulo y el de la magnitud. Finalmente entrega la nueva ecuación característica del sistema compensado y compara las respuestas al escalón.

La teoría completa y la verificación a mano están en [CASOS3.md](CASOS3.md).

## Requisitos

MATLAB o GNU Octave. **No requiere Control System Toolbox**: no se usan `tf()`, `rlocus()`, `step()`, `lsim()` ni `sisotool()`. Solo `poly()`, `roots()` y `ginput()`, que son funciones básicas del lenguaje.


## Uso

```matlab
>> proyecto3
```

```
   1) Ingresar ceros de la planta
   2) Ingresar polos de la planta
   3) Cargar una planta de ejemplo
   4) Ver G(s) en formato racional y ecuacion caracteristica
   5) Dibujar el lugar de las raices
   6) EDITAR polos/ceros con el raton (agregar, arrastrar, borrar)
   7) DISENAR compensador (click en la ubicacion deseada)
   8) Ver sistema compensado y nueva ecuacion caracteristica
   9) Comparar respuestas al escalon (planta vs compensada)
   0) Salir
```

### Prueba rápida

Opción `3` → `1` (planta `4/[s(s+2)]`), luego `7` → `3` (desde especificaciones) con `ζ = 0.5` y `ωₙ = 4`. Debe obtener el compensador clásico del libro de texto:

    Gc(s) = 4.732 (s + 2.928) / (s + 5.464)

Después, opción `8` para ver la nueva ecuación característica y `9` para comparar respuestas.

## Funcionalidad

**Entrada de ceros y polos** (opciones 1–3). Separados por comas; complejos con `i`, en pares conjugados. También se pueden cargar cinco plantas de ejemplo.

**Formato racional** (opción 4). Muestra `G(s)` tanto en forma factorizada como con los polinomios expandidos, junto con la ecuación característica `1 + K·G(s) = 0`, su grado y sus raíces actuales.

**Lugar de las raíces** (opción 5). Se calcula resolviendo la ecuación característica para ~2400 valores de `K`. Marca polos y ceros de la planta, los del compensador en otro color, las asíntotas con su centroide y la ubicación deseada. La ventana se ajusta a la geometría de cada sistema.

**Edición interactiva** (opción 6). Con el ratón sobre el gráfico:

- *agregar* polos o ceros con un click;
- *arrastrar* uno existente (primer click selecciona, segundo reubica);
- *borrar* el más cercano al click.

Los clicks cerca del eje real se ajustan a él automáticamente, y los que caen fuera generan el **par conjugado completo**, porque de otro modo los coeficientes del polinomio dejarían de ser reales. El lugar de las raíces se redibuja tras cada cambio. Si `ginput` no está disponible en el entorno, se piden las coordenadas por teclado.

**Diseño del compensador** (opción 7). La ubicación deseada `sd` se elige con el ratón, escribiendo coordenadas, o desde especificaciones de `ζ` y `ωₙ`. El programa muestra una tabla con el aporte de ángulo y magnitud de cada polo y cero en `sd`, calcula la **deficiencia angular** y ofrece cuatro tipos de compensador:

| Tipo | Forma | Cuándo |
|---|---|---|
| Ganancia pura | `Kc` | `sd` ya está sobre el lugar |
| PD | `Kc(s+zc)` | Falta ángulo; solución más simple |
| Adelanto | `Kc(s+zc)/(s+pc)` | Falta ángulo; método de la bisectriz |
| Atraso | `Kc(s+zc)/(s+pc)` | Mejorar el error estacionario sin alterar el lugar |

La ganancia `Kc` sale del criterio de la magnitud. El programa verifica que `sd` sea efectivamente raíz de la ecuación característica resultante y reporta el error.

**Sistema compensado** (opción 8). Muestra `G(s)`, `Gc(s)` y el lazo abierto compensado `Gc(s)G(s)` en forma factorizada y racional, la nueva ecuación característica con todos sus polos, un veredicto de estabilidad, un análisis de **dominancia** de los polos deseados y la constante de posición `Kp` con el error estacionario resultante.

**Comparación de respuestas** (opción 9). Respuesta al escalón del lazo cerrado sin compensar y compensado, integrando la ecuación de estado con **Runge-Kutta de cuarto orden** sobre la forma canónica controlable. Reporta valor final, sobreimpulso, tiempo de pico y tiempo de asentamiento al 2 % de cada una.

## Plantas de ejemplo

| # | `G(s)` | Particularidad |
|---|---|---|
| A | `4/[s(s+2)]` | Ejemplo canónico; reproduce el resultado del libro de texto |
| B | `1/[(s+1)(s+2)(s+3)]` | Tercer orden, lento |
| C | `1/[s(s+1)(s+5)]` | Con integrador y polo lejano |
| D | `1/[(s+0.5)(s+3)]` | Sin integrador: error estacionario apreciable |
| E | `1/[s(s−1)]` | **Planta inestable** que el compensador estabiliza |

## Validación de entradas

Los datos se leen como texto y se convierten numéricamente: una entrada inválida nunca detiene el programa, se indica el motivo y se vuelve a solicitar. Se exige al menos un polo, se verifica que los coeficientes resultantes sean reales, y en el diseño se comprueba que la deficiencia angular sea alcanzable por el tipo de compensador elegido antes de calcular nada.
