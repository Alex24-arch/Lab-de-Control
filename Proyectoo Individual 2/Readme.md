# EL-5409 — Proyecto individual 2

Análisis de estabilidad de un sistema realimentado: **tabla de Routh-Hurwitz** y **lugar de las raíces**, mediante una consola interactiva en MATLAB/Octave.

**Estudiante:** Santos Alexander López Valles **Carné:** 2023057479

## Descripción

A partir de los ceros y polos de la planta en lazo abierto `G(s) = Kg·num(s)/den(s)`, el script construye la ecuación característica de lazo cerrado

    1 + K·G(s) = 0        ⟺        den(s) + K·num(s) = 0

y determina la estabilidad del sistema por dos métodos independientes que se contrastan entre sí: la tabla de Routh-Hurwitz y el cálculo directo de las raíces.


## Archivos

| Archivo | Contenido |
|---|---|
| `script.m` | Script principal (consola interactiva) |
| `readme.md` | Teoría del método y explicación de los casos de prueba |

## Uso

Desde el directorio del repositorio:

```matlab
>> proyecto2
```

Se abre un menú que permanece activo hasta que se elige salir:

```
   1) Ingresar ceros de lazo abierto
   2) Ingresar polos de lazo abierto
   3) Ingresar ganancias (Kg de la planta, K del lazo)
   4) Cargar un caso de prueba
   5) Ver G(s) y la ecuacion caracteristica
   6) Tabla de Routh-Hurwitz (para el K actual)
   7) Rango de K que estabiliza el sistema
   8) Lugar de las raices (root locus)
   9) ANALISIS COMPLETO (5+6+7+8 y conclusion)
   0) Salir
```

### Formato de entrada

Ceros y polos se ingresan separados por comas o espacios; los complejos usan `i`:

```
  Polos: 0, -1, -2
  Polos: 0, -1+1i, -1-1i
  Ceros: [ENTER para dejar vacío si la planta no tiene ceros]
```

Las raíces complejas deben ingresarse **en pares conjugados**. Los polos son obligatorios; los ceros son opcionales.

### Prueba rápida

Opción `4` → `1`, luego opción `9`. El sistema `G(s) = 1/[s(s+1)(s+2)]` debe reportarse estable con `K = 1`, con `K` crítico igual a 6 y cruce del eje imaginario en `s = ±j1.414`.

## Funcionalidad

**Ecuación característica.** Se construye `den(s) + K·num(s)` a partir de los ceros y polos y se muestra en forma polinomial legible, junto con el grado y el número de polos de lazo cerrado.

**Tabla de Routh-Hurwitz.** Se construye para el grado que corresponda. Antes se verifica la condición necesaria (todos los coeficientes presentes y del mismo signo). El número de cambios de signo en la primera columna da la cantidad de polos en el semiplano derecho. Se contemplan los dos casos especiales de la teoría:

- primer elemento de una fila igual a cero → se sustituye por un `ε` pequeño;
- fila completa de ceros → se reemplaza por la derivada del polinomio auxiliar.

**Verificación cruzada.** Las raíces se calculan además con `roots()` y se comparan con el resultado de la tabla; el programa marca `[OK]` cuando ambos métodos coinciden.

**Rango de K.** Barrido numérico con refinamiento por bisección que reporta los valores de `K` donde el sistema cambia de estable a inestable, junto con la frecuencia de cruce del eje imaginario.

**Lugar de las raíces.** Se calcula resolviendo la ecuación característica para ~3000 valores de `K`. El gráfico muestra las ramas, los polos de lazo abierto (**×** rojas) donde nacen en `K = 0`, los ceros (**○** verdes) donde terminan cuando `K → ∞`, las asíntotas de las `n−m` ramas restantes con su centroide, y los cruces del eje imaginario (**★** naranja) correspondientes al `K` crítico. La ventana del gráfico se ajusta automáticamente a la geometría de polos y ceros de cada sistema, y el barrido de `K` se detiene cuando las ramas salen de esa ventana.

**Conclusión.** Veredicto de estabilidad justificado, indicando cuántos polos quedan en el semiplano derecho y qué implica eso para la respuesta del sistema.

## Casos de prueba incluidos

| # | `G(s)` | Rango estable | Qué demuestra |
|---|---|---|---|
| A | `1/[s(s+1)(s+2)]` | 0 < K < 6 | Caso clásico; en `K = 6` aparece el caso especial de fila nula |
| B | `1/[(s+1)(s+2)(s+3)]` | 0 < K < 60 | Planta estable en lazo abierto que se desestabiliza al subir K |
| C | `(s+2)/[s(s+1)]` | todo K > 0 | Efecto estabilizador de un cero |
| D | `1/[s(s²+2s+2)]` | 0 < K < 4 | Manejo de polos complejos conjugados |
| E | `1/[s(s−1)]` | ninguno | Ningún K estabiliza: falla la condición necesaria |

## Validación de entradas

Los datos se leen como texto y se convierten numéricamente, de modo que una entrada inválida nunca detiene el programa: se indica el motivo y se vuelve a solicitar el valor. Se rechazan entradas no numéricas, infinitos y `NaN`; se exige al menos un polo; se verifica que los coeficientes resultantes sean reales (raíces complejas en pares conjugados) y se detecta el caso en que la ecuación característica degenera.
