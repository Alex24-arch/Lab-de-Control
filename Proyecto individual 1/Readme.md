# EL-5409 — Proyecto individual 1

Simulación paramétrica de una planta de primer orden (motor de CD), con una **consola interactiva** en MATLAB.

**Estudiante:** Santos Alexander López Valles — **Carné:** 2023057479

## Modelo

    G(s) = KM / (tau*s + 1)

    KM  = Kt / (Ra*b + Kt*Kb)
    tau = Ra*J / (Ra*b + Kt*Kb)

| Parámetro | Descripción | Unidades |
|---|---|---|
| `Kt` | Constante de par del motor | N·m/A |
| `Ra` | Resistencia de armadura | Ω |
| `b`  | Coeficiente de fricción del eje | N·m·s/rad |
| `Kb` | Constante de fuerza electromotriz | V·s/rad |
| `J`  | Momento de inercia (motor + carga) | kg·m² |

La explicación completa del modelo, de dónde sale cada valor del gráfico y qué demuestra cada caso de prueba está en [CASOS.md](CASOS.md).

## Archivos

| Archivo | Contenido |
|---|---|
| `script.m` | **Script principal.** Aplicación de consola interactiva. |
| `CASOS.md` | Documentación: teoría del modelo y explicación de los 5 casos de prueba. |
| `README.md` | Este archivo. |

Compatible con **MATLAB** y **GNU Octave**. No requiere Control System Toolbox: la respuesta se calcula analíticamente con `y(t) = KM(1 − e^(−t/τ))`, no con `step()`.

## Uso

Desde el directorio del repositorio:

```matlab
>> proyecto1
```

Se abre un menú que permanece activo hasta que se elige salir:

```
   1) Ingresar todos los parametros
   2) Modificar un parametro
   3) Cargar un caso de prueba
   4) Ver parametros actuales
   5) Simular y graficar respuesta al escalon
   6) Exportar resultados (CSV + PNG)
   0) Salir
```

- El menú muestra siempre el estado actual de los cinco parámetros.
- Al ingresar datos, **ENTER** conserva el valor actual.
- La opción 2 permite cambiar un solo parámetro y volver a simular, para aislar el efecto de cada uno.
- La opción 3 carga cualquiera de los cinco casos de prueba sin tener que teclear valores.

### Prueba rápida

Opción `3` → `5` (caso demostrativo) → opción `5` (simular). Debe obtener `KM = 0.857143` y `τ = 0.5 s`.

## Casos de prueba incluidos

| # | Caso | `KM` | `tau` [s] | `ess` | Qué demuestra |
|---|---|---|---|---|---|
| A | Motor didáctico | 0.0999 | 0.0999 | 0.9001 | `ess` grande, domina el gráfico |
| B | Error pequeño pero visible | 0.9524 | 0.9524 | 0.0476 | `ess` justo sobre el umbral del 2 % |
| C | Seguimiento perfecto | 1.0000 | 0.0500 | 0.0000 | `ess = 0`, rama alternativa del código |
| D | Motor real (Maxon RE-25) | 42.5547 | 0.0451 | −41.5547 | `KM >> 1`, autoescalado de ejes |
| E | Caso demostrativo | 0.8571 | 0.5000 | 0.1429 | todas las marcas bien separadas |

## Validación de entradas

Cada dato se lee como texto y se convierte con `str2double`, de modo que una entrada no numérica nunca detiene el programa: se muestra el motivo y se vuelve a pedir el valor.

Se verifica que:

- todos los parámetros sean escalares reales y finitos;
- `Kt`, `Ra` y `J` sean estrictamente positivos (un motor sin par, sin resistencia o sin inercia no tiene sentido físico);
- `b` y `Kb` no sean negativos;
- el denominador `Ra*b + Kt*Kb` sea mayor que cero — `b` y `Kb` no pueden ser ambos cero, porque sin amortiguamiento el modelo dejaría de ser de primer orden.

## Salidas

**Consola:** parámetros de entrada, el denominador común, los coeficientes `KM` y `τ`, la función de transferencia resultante y todos los valores característicos de la respuesta.

**Gráfico:** respuesta al escalón unitario en el dominio del tiempo, con:

- entrada escalón unitario (referencia);
- valor final esperado, con marca en `t = 5τ`;
- banda de ±2 % del valor final;
- valor de la respuesta en `t = τ` (63.2 % del valor final);
- tiempo de asentamiento al 2 %, `ts = 4τ`;
- error de estado estacionario `ess = 1 − KM`, acotado en el gráfico cuando es visible en la escala.

**Exportación (opción 6):** archivo `.csv` con la curva `t, y` y un encabezado con los parámetros usados, y archivo `.png` con la figura.
