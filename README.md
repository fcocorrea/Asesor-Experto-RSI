# Asesor Experto: Estrategia RSI con Confirmación de EMA

Este Asesor Experto (EA) para **MetaTrader 5** implementa una estrategia de trading que busca operar a favor de la tendencia principal, utilizando el **RSI** para identificar condiciones extremas de mercado (sobreventa/sobrecompra) como disparador inicial, y un **cruce de Medias Móviles Exponenciales (EMAs)** como confirmación de entrada.

## 📈 Estrategia de Trading

La lógica central del EA es esperar a que el mercado muestre un posible agotamiento a través del RSI para luego entrar en la dirección de la nueva tendencia confirmada por las EMAs.

### Lógica de Compra (Long)
Una operación de compra se ejecuta cuando se cumplen las siguientes condiciones en orden:
1.  **Disparador RSI:** El indicador RSI de 14 períodos cae por debajo del nivel de sobreventa definido por el usuario (ej: `27`). Esto activa una "bandera" (`RSI_flag_compra`) que pone al EA en modo de búsqueda de compra.
2.  **Confirmación de Tendencia:** Se produce un cruce alcista de las EMAs (la EMA corta cruza por encima de la EMA larga).
3.  **Filtros de Calidad:**
    * **Distancia del Máximo Histórico (ATH):** El precio actual debe estar a una distancia porcentual mínima por debajo del ATH, evitando comprar en picos extremos.
    * **Distancia del Último Cruce Bajista:** El precio debe estar suficientemente alejado (medido en múltiplos de ATR) del último cruce bajista de EMAs para confirmar que la nueva tendencia alcista tiene espacio para desarrollarse.

### Lógica de Venta (Short)
Una operación de venta se ejecuta cuando se cumplen las siguientes condiciones en orden:
1.  **Disparador RSI:** El indicador RSI de 14 períodos sube por encima del nivel de sobrecompra definido por el usuario (ej: `73`). Esto activa una "bandera" (`RSI_flag_venta`) para buscar una venta.
2.  **Confirmación de Tendencia:** Se produce un cruce bajista de las EMAs (la EMA corta cruza por debajo de la EMA larga).
3.  **Filtro de Calidad:**
    * **Distancia del Último Cruce Alcista:** El precio debe estar suficientemente alejado (medido en múltiplos de ATR) del último cruce alcista de EMAs.

---

## 🛡️ Gestión de Riesgo y de Posiciones

El EA incluye un sistema de gestión de riesgo robusto y configurable para proteger el capital y asegurar las ganancias.

* **Cálculo de Volumen:** El tamaño de la posición se calcula automáticamente en cada operación para arriesgar un porcentaje fijo del balance de la cuenta (ej: `5%`), basado en la distancia entre el precio de entrada y el Stop Loss.
* **Stop Loss (SL):**
    * Se calcula basándose en el último precio extremo relevante (mínimo local para compras, máximo local para ventas) más un múltiplo del ATR para darle espacio a la operación.
    * **Puede desactivarse.** Si `activarStopLoss` es `false`, el único mecanismo de cierre por pérdida será el cruce de EMAs en contra.
* **Take Profit (TP):**
    * Se calcula en base a un ratio Riesgo/Beneficio (RR) definido por el usuario. Por ejemplo, con un `ratioRR` de 2, el TP se colocará al doble de la distancia del SL.
* **Break Even:** Mueve el Stop Loss al precio de entrada una vez que la operación ha avanzado a favor una distancia determinada por un múltiplo de ATR. Esto elimina el riesgo de la operación.
* **Trailing Stop:** Una vez la operación está en ganancia, el Stop Loss se mueve dinámicamente siguiendo el precio a una distancia definida por un múltiplo de ATR, asegurando beneficios a medida que el mercado avanza a favor.
* **Cierre por Cruce de EMA:** El EA permite usar un cruce de EMAs en contra como señal de cierre.
    * Si el SL fijo está **desactivado**, este es el único método de stop.
    * Si el SL fijo está **activado**, esta opción puede usarse para cerrar operaciones en ganancia antes de que lleguen al TP o al SL.

---

## ⚙️ Parámetros de Entrada (Inputs)

Todos los aspectos del EA pueden ser personalizados a través de los siguientes parámetros de entrada para optimizar su rendimiento en diferentes activos y temporalidades.

| Parámetro                                       | Valor por Defecto | Descripción                                                                                                    |
| ----------------------------------------------- | ----------------- | -------------------------------------------------------------------------------------------------------------- |
| **--- AJUSTES GENERALES PARA INDICADORES ---** |                   |                                                                                                                |
| `barrasEMAlarga`                                | `26`              | Periodo de la Media Móvil Exponencial lenta, usada para definir la tendencia principal.                        |
| `barrasEMAcorta`                                | `9`               | Periodo de la Media Móvil Exponencial rápida, usada para las señales de cruce.                                 |
| **--- AJUSTES PARA COMPRAS (LONG) ---** |                   |                                                                                                                |
| `nivelDeActivacionRSI_Compra`                   | `27`              | Nivel de RSI por debajo del cual se activa la búsqueda de una señal de compra (condición de sobreventa).       |
| `lejaniaConATH_Compra`                          | `2.5`             | Distancia porcentual mínima que debe tener el precio por debajo del ATH para validar una compra.               |
| `lejaniaConUltimoCruceEmaHaciaAbajo_Compra`     | `3`               | Múltiplo de ATR que define la distancia mínima al último cruce bajista para validar una compra.                |
| **--- AJUSTES PARA VENTAS (SHORT) ---** |                   |                                                                                                                |
| `nivelDeActivacionRSI_Venta`                    | `73`              | Nivel de RSI por encima del cual se activa la búsqueda de una señal de venta (condición de sobrecompra).        |
| `lejaniaConUltimoCruceEmaHaciaArriba_Venta`     | `3`               | Múltiplo de ATR que define la distancia mínima al último cruce alcista para validar una venta.                 |
| **--- AJUSTES GENERALES PARA GESTIONAR OPERACIÓN ---** |            |                                                                                                                |
| `activarStopLoss`                               | `true`            | Si es `true`, se utiliza un Stop Loss fijo. Si es `false`, la operación se cierra por cruce de EMAs.            |
| `activarTrailingStop`                           | `true`            | Activa o desactiva la función de Trailing Stop.                                                                |
| `activarBreakEven`                              | `true`            | Activa o desactiva la función de Break Even.                                                                   |
| `activarTakeProfit`                             | `true`            | Activa o desactiva el Take Profit fijo basado en el ratio RR.                                                  |
| `activarCruceEMA`                               | `true`            | Si `activarStopLoss` es `true`, permite cerrar operaciones en ganancia por cruce de EMA.                         |
| `multiplicadorATRDesdePrecioExtremo`            | `3`               | Múltiplo de ATR para calcular la distancia del Stop Loss desde el último precio extremo (mínimo/máximo).      |
| `Compra_multiplicadorATR_TrailingStop`          | `3`               | Múltiplo de ATR para la distancia del Trailing Stop en operaciones de compra.                                  |
| `Compra_multiplicadorATR_BreakEven`             | `1`               | Múltiplo de ATR para la distancia a la que se activa el Break Even en compras.                                 |
| `Venta_multiplicadorATR_TrailingStop`           | `3`               | Múltiplo de ATR para la distancia del Trailing Stop en operaciones de venta.                                   |
| `Venta_multiplicadorATR_BreakEven`              | `1`               | Múltiplo de ATR para la distancia a la que se activa el Break Even en ventas.                                  |
| `ratioRR`                                       | `2`               | Ratio Riesgo/Beneficio para el cálculo del Take Profit (ej: 2 significa 2:1).                                  |
| `riesgoPorOperacion`                            | `5`               | Porcentaje del balance de la cuenta a arriesgar en cada operación.                                             |

---

## 🛠️ Instalación y Uso

1.  Abre **MetaEditor** desde tu terminal de MetaTrader 5.
2.  Crea un nuevo Asesor Experto (`.mq5`).
3.  Copia y pega el código fuente en el archivo.
4.  Haz clic en **Compilar** (o presiona F7). Si no hay errores, el EA estará listo.
5.  En la ventana "Navegador" de MetaTrader 5, busca tu EA en la sección "Asesores Expertos".
6.  Arrastra el EA a un gráfico.
7.  Configura los parámetros de entrada en la ventana que aparece y asegúrate de que el "Trading Automatizado" esté activado.
8.  Haz clic en "Aceptar".

---

## ⚠️ Descargo de Responsabilidad

El trading automatizado conlleva un riesgo significativo. El rendimiento pasado no es garantía de resultados futuros. Realiza siempre tus propias pruebas (backtesting) y utiliza este software bajo tu propio riesgo. Se recomienda encarecidamente probar el EA en una cuenta demo antes de usarlo en una cuenta real.