---
title: Aprender — ruta de nodos adaptativos
tags:
  - paessed
  - producto
  - aprender
  - ui-ux
actualizado: 2026-11-07
estado: diseño en definición
---

# Aprender — ruta de nodos adaptativos

> **Estado:** especificación de trabajo, no implementación terminada. Se distinguen decisiones conversadas, propuestas de algoritmo y dudas abiertas. No modifica el contrato `.paessed` ni las decisiones históricas registradas en `docs/DECISIONS.md`.

## 1. Objetivo

Crear una ruta de práctica de Matemática M1 inspirada visualmente en Duolingo. Tras un diagnóstico de nivelamiento, PAESSED recomienda **nodos de cinco preguntas** obtenidas del banco central de ensayos DEMRE. La ruta se reajusta con el historial de respuestas para reforzar los subejes donde el estudiante muestra más dificultades, sin abandonar los demás.

**La función principal del nodo es practicar**, no representar una clase teórica, un nivel de dificultad ni un desbloqueo obligatorio. La teoría guiada podría estudiarse después como funcionalidad separada.

### Flujo previsto

1. El estudiante elige cuánto tiempo desea dedicar diariamente.
2. Realiza un diagnóstico de nivelamiento y recibe un perfil inicial de dominio.
3. PAESSED genera los primeros nodos de su ruta.
4. Cada nodo presenta cinco preguntas de un subeje M1, con el eje visible.
5. Al terminar, el sistema guarda respuestas, muestra revisión y recalcula el perfil.
6. Los próximos nodos pueden cambiar según la nueva evidencia; los ya completados no cambian.

## 2. Decisiones de producto ya conversadas

| Elemento | Dirección actual |
| --- | --- |
| Formato | Ruta visual de nodos tipo Duolingo, integrada en la UI existente de PAESSED. |
| Tamaño | **5 preguntas por nodo**. |
| Banco | Banco central *Default* de ensayos DEMRE, con preguntas clasificadas y aptas para uso en Aprender. |
| Clasificación | **4 ejes y 16 subejes M1**, usando `eje` y `subeje` del contrato `.paessed` 0.2.0. |
| Personalización | **Mantenimiento:** rotación equitativa entre los cuatro ejes. **Refuerzo:** rotación dinámica entre uno o varios ejes y subejes de menor dominio. |
| Proporción semanal | **4 nodos de mantenimiento por cada 3 de refuerzo**, intercalados. |
| Ubicación de refuerzos | Posiciones con aleatoriedad controlada; cuotas exactas y ruta guardada. |
| Dominio | Respuestas originales válidas, procedentes de Aprender, Playground y Ensayos; ponderación por nuevas respuestas del mismo subeje, no por días. |
| Calibración | **15 respuestas válidas por eje** para dejar el estado gris provisional; sin cobertura forzosa de cada subeje. |
| HLR | **Fuera del MVP.** Se conserva solo como referencia histórica/posible investigación futura; no participa en mantenimiento, refuerzo ni dominio. |
| Frecuencia | Recalcular los grupos de refuerzo tras nuevas respuestas válidas y actualizar **solo los nodos futuros**; guardar el cursor de rotación y conservar el progreso. |
| Gamificación | Racha, XP y progreso visual pueden motivar; **XP no equivale a dominio matemático**. |
| Exclusiones | No usar los campos `habilidad` ni `dificultad`: no forman parte del contrato vigente 0.2.0. |
| Ranking | Separado de Aprender. No condiciona su primer MVP. |

### Preferencia de dedicación

| Elección del usuario | Nodos diarios | Preguntas diarias | Nodos en 7 días, si cumple cada día |
| --- | ---: | ---: | ---: |
| Poco tiempo | 1 | 5 | 7 |
| Normal | 2 | 10 | 14 |
| Más tiempo | 3 | 15 | 21 |

La meta es un **objetivo**, no un límite: falta decidir si se ofrece práctica adicional una vez completada. El tiempo real de resolución varía según las preguntas; no prometer una duración fija por nodo.

**Cambio respecto a una idea anterior:** ya no se propone una cuota fija universal de 16 preguntas diarias. Se adopta esta escala de 5/10/15 según la elección del estudiante, pendiente de validación en el piloto.

## 3. Planificación semanal y selección adaptativa

### 3.1. Cuántos nodos y dónde aparecen

**Decisión:** por cada **7 nodos**, programar **4 de mantenimiento (M)** y **3 de refuerzo (R)**. Ambos se intercalan a lo largo de la semana, en vez de reservar días exclusivos para cada tipo.

| Preferencia | Nodos en siete días completos | Mantenimiento | Refuerzo |
| --- | ---: | ---: | ---: |
| Poco tiempo (1/día) | 7 | 4 | 3 |
| Normal (2/día) | 14 | 8 | 6 |
| Más tiempo (3/día) | 21 | 12 | 9 |

- **Azar controlado solo para la posición de M/R.** Generar y guardar un calendario con las cuotas exactas; no sortear los ejes ni subejes de refuerzo.
- En 3 nodos diarios: **cinco días con 2 M + 1 R** y **dos días con 1 M + 2 R**, con orden variable. En 2/día: seis días con 1 M + 1 R y uno con 2 M. En 1/día: cuatro jornadas M y tres R.
- La ruta generada debe ser **estable y reproducible**; no reorganizar lo ya completado, ni cambiar nodos únicamente por recargar la página.
- Son metas programadas, no resultados obtenidos. Si el estudiante omite días, no contar como completados sus nodos. La replanificación de días omitidos, cambios de dedicación y práctica extra sigue pendiente.

### 3.2. Nodos de mantenimiento: cobertura equitativa

**Decisión:** distribuir los nodos M **equitativamente entre los cuatro ejes**, sin priorizar por dominio ni por HLR.

| Dedicación | Nodos M por eje y semana completa |
| --- | ---: |
| 1 nodo/día | 1 |
| 2 nodos/día | 2 |
| 3 nodos/día | 3 |

- Rotar por los cuatro ejes con un cursor persistido y completar la cuota de cada eje.
- **Propuesta para los subejes del eje asignado:** rotación equitativa de los subejes con preguntas elegibles, manteniendo progreso entre semanas. El orden concreto y la política de escasez de preguntas aún deben probarse.
- Esto garantiza la cobertura de **ejes** cuando se completa la planificación; no promete cubrir todos los subejes en una sola semana, ni compensa automáticamente los días omitidos.

### 3.3. Nodos de refuerzo: selección de ejes por dominio

**Dirección acordada:** el refuerzo **no se sortea** ni se asigna por HLR. Se basa en el dominio de los ejes y subejes, dando espacio a varias debilidades mediante **rotación**.

Propuesta de selección, **pendiente de simulación y ajuste de umbrales**:

1. Ordenar los cuatro ejes **de menor a mayor dominio**.
2. Si entre el peor y el segundo hay **30 puntos porcentuales o más de diferencia**, proponer **solo el eje más débil**.
3. En caso contrario, partir de **los dos ejes más débiles**.
4. Incorporar el tercero y el cuarto si sus dominios están a **10 puntos porcentuales o menos del eje más débil**. Así podrían rotar **1, 2, 3 o 4 ejes**, según lo parecidos que sean sus resultados.
5. Rotar los ejes elegidos, **empezando por el más débil**, sin repetir eje consecutivo mientras haya otros seleccionados. **Si solo hay uno claramente débil, es correcto repetir ese eje en nodos R.**

Estos valores de **10 y 30 puntos son hipótesis de diseño**, no parámetros aprobados científicamente. No se debe tratar una diferencia pequeña o datos en calibración como prueba definitiva de que un eje es peor.

| Ejemplo de dominios por eje (orden ascendente) | Resultado inicial propuesto |
| --- | --- |
| 38 %, 64 %, 88 %, 92 % | **2 ejes:** 38 % y 64 %; alternar ambos. |
| 38 %, 85 %, 88 %, 92 % | **1 eje:** 38 %; concentrar allí los refuerzos. |
| 40 %, 42 %, 48 %, 90 % | **3 ejes:** 40 %, 42 % y 48 %; rotarlos. |
| 40 %, 42 %, 48 %, 49 % | **4 ejes:** valores próximos; rotar todos. |

**Ejemplo ilustrativo** con 14 nodos por semana y ejes débiles Geometría y Probabilidad:

| Eje | Mantenimiento | Refuerzo | Total |
| --- | ---: | ---: | ---: |
| Geometría | 2 | 3 | 5 |
| Probabilidad y Estadística | 2 | 3 | 5 |
| Álgebra y Funciones | 2 | 0 | 2 |
| Números | 2 | 0 | 2 |
| **Total** | **8** | **6** | **14** |

Se trata de una **rotación equitativa dentro del grupo de refuerzo elegido**, no de una cuota por eje que dependa directamente de la magnitud del déficit.

### 3.4. Dentro del eje de refuerzo: elegir el subeje

- Aplicar la **misma idea de selección y rotación por menor dominio** entre los subejes del eje seleccionado, usando solo aquellos con preguntas válidas.
- Probar umbrales de proximidad antes de fijarlos: los ejes tienen distinta cantidad de subejes y la evidencia por subeje puede ser escasa.
- La rotación impide que el algoritmo elija **siempre** el mismo subeje cuando existen varios débiles. **No prohíbe repetirlo** si es el único que necesita refuerzo.
- **HLR no interviene**. Si algunos subejes todavía no tienen evidencia suficiente, hace falta una regla de exploración/calibración que no interprete «sin datos» como 0 % de dominio.

### 3.5. Recalcular sin perder progreso

- Cuando entran nuevas respuestas **originales válidas**, actualizar el dominio y **revaluar el grupo de ejes y subejes elegibles para refuerzo**.
- Los nodos **ya completados y sus primeros intentos son inmutables**. La selección nueva afecta solo a los **nodos futuros aún no iniciados**.
- Guardar cursor, posición y versión de planificación para evitar cambios arbitrarios al recargar.
- Si un eje mejora claramente (por ejemplo, Geometría pasa de 38 % a 75 %), el sistema puede dejar de incluirlo en refuerzo y sustituirlo por otro eje que ahora sea más débil.
- La frecuencia de recálculo, cómo tratar empates y cuándo congelar un nodo «próximo» son decisiones de implementación que requieren pruebas.

## 4. Calibración y dominio (sin HLR en el MVP)

### Período de calibración del dominio

- Mostrar un **porcentaje provisional en gris** mientras el eje tenga menos de **15 preguntas originales válidas** respondidas.
- A partir de **15 por eje**, habilitar sus colores de dominio (Crítico, Débil, Regular, Fuerte, Dominado), pero presentarlo todavía como **estimación**, nunca como conocimiento verdadero o puntaje PAES.
- **No exigir cobertura obligatoria de todos los subejes** para terminar la calibración; sí observar si una concentración excesiva en uno solo vuelve poco representativo al eje.
- La calibración se evalúa por eje, no como una barrera global: diferentes ejes pueden completarla en momentos distintos.
- El tamaño del **diagnóstico inicial** todavía no está reconciliado: `docs/DECISIONS.md` registra 12 preguntas y en conversaciones posteriores se propusieron 16 (una por subeje). **No confundir diagnóstico con las 15 evidencias por eje**, que pueden acumularse entre actividades.

### Qué respuestas aportan evidencia

Unificar los intentos elegibles de **Aprender, Playground y Ensayos**. El sistema debe registrar para cada pregunta: identidad estable, eje/subeje, origen, momento, respuesta del primer intento, si recibió ayuda y si ya había visto la pregunta.

- Contabilizar solo **primeros intentos originales, sin ayudas y de preguntas nuevas para ese estudiante**.
- No aumentar dominio ni XP por reintentos de la ronda de corrección, ni por responder una pregunta cuya clave ya vio.
- Evitar doble conteo si la misma pregunta aparece después en otro modo o ensayo.
- Mantener todo el historial para trazabilidad; no resetearlo al iniciar una nueva semana.
- La combinación exacta de resultados de diferentes actividades se debe verificar, ya que las condiciones de una prueba cronometrada no son idénticas a las de práctica libre.

### Cálculo propuesto: ponderación por nuevas respuestas, NO por días

**Cambio importante respecto de ideas anteriores:** no reducir automáticamente el dominio por no entrar a la web ni por dejar un eje sin practicar. El paso de los días, por sí solo, **no modifica el dominio**.

Propuesta de peso de cada respuesta válida de un subeje:

~~~math
w_i = 0.95^{k_i/5}
D_s = 100 * (Σ_i w_i a_i) / (Σ_i w_i)
~~~

- `a_i = 1` si acertó en su primer intento, `0` si falló.
- `k_i` = número de **respuestas válidas más recientes del mismo subeje** acumuladas después de esa respuesta.
- Cada cinco respuestas nuevas del subeje reducen el peso anterior aproximadamente a **95 % de su valor previo**.
- Una respuesta recién realizada pesa **100 %**. Si pasan diez días sin responder ese subeje, **su peso no cambia**.
- Recalcular el porcentaje al llegar respuestas válidas nuevas. El historial permanece completo aunque las antiguas influyan menos.
- Para un eje, combinar los subejes con suficiente evidencia; **la regla exacta de agregación por eje queda pendiente** para no dejar que un subeje sobrepracticado represente a todos.

**El factor 0,95 y el bloque de cinco son parámetros propuestos, no validados.** Se deben comparar con otras ponderaciones y ventanas de observación antes de dar el modelo por definitivo.

#### Ideas exploradas que ya NO son la dirección actual del dominio

- **`(C+2)/(N+4)`**: suavizado Beta-Binomial real, pero el `+2/+4` fue un valor elegido arbitrariamente y se prefirió eliminarlo en favor de calibración explícita.
- **Solo últimas 20 respuestas**: ejemplo provisional; la dirección actual es ponderar las respuestas según intentos nuevos, manteniendo el historial.
- **Curva por antigüedad calendario de 30 días**, `w(d) = max(0,1-d/30)^1.5`: se exploró, pero se identificó que podría penalizar injustamente a quien no estudia un día o a un eje que tarda en reaparecer. **No aplicarla al dominio.**
- **Resetear todas las estadísticas cada siete días:** solo se renueva la planificación y los indicadores semanales, **no el historial ni el dominio acumulado**.

### HLR: referencia descartada del MVP, no una dependencia

HLR (*Half-Life Regression*) es un modelo de repaso espaciado que estima la probabilidad de recordar, según tiempo y semivida de memoria:

~~~math
P(recordar) = 2^(-t/h)
~~~

En el diseño inicial de Aprender **no lo necesitamos**: mantenimiento ya cubre equitativamente los cuatro ejes, y refuerzo prioriza el menor dominio. Por eso:

- **No usar HLR** para decidir qué M o R aparece, elegir ejes/subejes, ni alterar el dominio.
- **No introducir una ponderación dominio/HLR (p. ej., 70/30)**: fue una propuesta conversada y se descartó por duplicar responsabilidades.
- Conservar HLR **solo como referencia histórica o investigación futura** si las pruebas evidencian que la rotación de mantenimiento resulta insuficiente. Sus parámetros no están calibrados para M1.
- Referencia: [Duolingo — Half-Life Regression](https://research.duolingo.com/papers/settles.acl16.pdf).

### Separación de responsabilidades vigente

| Componente | Qué hace |
| --- | --- |
| **Planificador 4:3** | Define cuotas semanales y posiciones intercaladas M/R; solo estas posiciones tienen azar controlado. |
| **Mantenimiento** | Rota equitativamente los cuatro ejes y, como propuesta, los subejes del eje correspondiente. |
| **Estimador de dominio** | Calcula resultados ponderados por nuevas respuestas válidas, sin penalizar inactividad. |
| **Refuerzo** | Selecciona los ejes y subejes más débiles y rota dinámicamente entre los elegidos, con umbrales aún provisionales. |

**No se trata de cuatro algoritmos probabilísticos.** La selección del contenido es determinista, sujeta a cambios cuando llegan nuevas respuestas y a reglas que faltan validar.

## 5. Banco central y selección de preguntas

- Utilizar la fuente *Default* central de preguntas importadas desde ensayos DEMRE, con su procedencia identificable.
- Elegir preguntas válidas, etiquetadas con `eje` y `subeje` y, cuando corresponda, `en_temario`.
- Conservar identidad de ensayo y pregunta para detectar duplicados y evitar repetirlas demasiado pronto.
- No confundir aprobación **Gold** de fidelidad con permiso para publicación: el contenido debe cumplir también las reglas de derechos y publicación de [[sistema/BANCO_PREGUNTAS]].
- No usar preguntas incompletas, sin alternativas utilizables o con clave no acreditada para una sesión evaluada.
- Si un subeje no tiene cinco preguntas disponibles, **no inventarlas ni etiquetar como ese subeje ejercicios de otro**. Definir un fallback visible y honesto antes de habilitarlo.
- La clasificación de contenido y la lógica del algoritmo deben ser independientes del importador; **no agregar campos al contrato 0.2.0**.

## 6. UI/UX propuesta

### Pantalla Aprender

- Encabezado con progreso diario, racha y acceso a ajustes de dedicación.
- Mostrar nodos **Mantenimiento** y **Refuerzo** intercalados; no fijar cuatro días normales y tres exclusivos de refuerzo.
- Mostrar por eje «En calibración» en gris y progreso de respuestas originales, por ejemplo «8/15», hasta alcanzar la evidencia mínima.
- Ruta de nodos grandes, claros y táctiles, con conexión visual estilo Duolingo **sin copiar literalmente su diseño**.
- Cada nodo muestra **eje, subeje y cinco preguntas**; el estado puede ser completado, disponible o futuro.
- Señalar «Refuerzo» o «Mantenimiento» según corresponda, sin tratarlo como una dificultad de la pregunta.
- Mostrar progreso por ejes y explicar brevemente la selección (p. ej., «Mantenimiento: turno de Geometría» o «Refuerzo: subeje de menor dominio»).
- Mantener la identidad visual de PAESSED y reutilizar el visor de preguntas a pantalla completa de Ensayos/Playground cuando sea viable.
- Preparar escritorio, móvil, teclado, accesibilidad y movimiento reducido.

### Ruta dinámica

- Los nodos **completados** permanecen en el historial.
- Los nodos **futuros no iniciados** son previsiones y pueden cambiar de eje/subeje cuando se recalcula el dominio; los completados no se reescriben.
- No confundir un cambio de ruta con la pérdida de progreso.
- Pendiente: cuántos nodos futuros se deben mostrar y qué animación explica un cambio de posición sin desconcertar al estudiante.

### Al terminar cada nodo: corrección de errores

**Comportamiento acordado para el diseño:**

1. El estudiante responde las **cinco preguntas** del nodo en una primera ronda.
2. Se guardan los resultados originales de cada pregunta (correcta, incorrecta u omitida, según las reglas de evaluación que se definan). **Solo esta primera ronda aporta evidencia de aciertos/errores al algoritmo adaptativo.**
3. Si no hubo errores, se pasa directamente a «Nodo completado».
4. Si hubo errores, aparece un resumen de la primera ronda y una acción clara: **«Corregir mis errores»**.
5. Se vuelven a mostrar únicamente las preguntas falladas para que el estudiante las responda correctamente, con acceso a una explicación o ayuda pedagógica.
6. Cuando las preguntas pendientes estén corregidas, se muestra **«Nodo completado»**, el progreso diario y el siguiente nodo recomendado.

**Ejemplo:** cinco preguntas, tres correctas y dos incorrectas en el primer intento. La evaluación queda en **3/5 (60 %)**. Si el estudiante corrige después las otras dos, el nodo se marca como completado y se muestra «2 errores corregidos», **sin reemplazar 3/5 por 5/5 en las estadísticas de dominio**.

#### Separación de datos y reglas

- Conservar el **primer intento** de forma inmutable para el cálculo del perfil; no convertir un acierto en el reintento en un acierto original.
- Registrar por separado los reintentos y el estado educativo de corrección/completitud.
- Los reintentos **no cuentan como preguntas nuevas** para la adaptación, cobertura, porcentaje de aciertos ni XP repetible.
- Mantener las respuestas y el progreso de corrección si el estudiante sale o recarga la página.
- Evitar bloqueos indefinidos: queda pendiente definir cuántos intentos fallidos habilitan una resolución guiada o la explicación completa y cómo se da por revisada la pregunta sin simular que fue correcta.
- No estimar puntaje PAES a partir de solo cinco preguntas.

#### Pruebas de aceptación específicas

- [ ] Cinco aciertos originales: completar sin ronda de corrección.
- [ ] Tres aciertos y dos errores: corregir solo esas dos; conservar 3/5 en dominio.
- [ ] Fallar nuevamente una pregunta: permitir revisión sin sobrescribir el primer intento.
- [ ] Cerrar y reabrir en plena corrección: reanudar sin duplicar resultados.
- [ ] Completar la corrección: habilitar el siguiente nodo una sola vez y sin duplicar XP.
- [ ] Verificar que los reintentos no alteren la selección adaptativa del siguiente nodo.

## 7. Pruebas de simulación requeridas antes de integrar

- [ ] Simular 30, 60 y 90 días con perfiles débiles, equilibrados y avanzados.
- [ ] Probar las tres metas diarias (1, 2 y 3 nodos).
- [ ] Verificar que cada eje tenga cobertura semanal cuando existen suficientes sesiones y preguntas elegibles.
- [ ] Verificar que ningún subeje con contenido disponible desaparezca indefinidamente.
- [ ] Detectar concentraciones excesivas y repeticiones de subeje o preguntas.
- [ ] Recalcular cuotas tras cambios de resultados sin alterar nodos completados.
- [ ] Comprobar resultados deterministas con los mismos datos de entrada.
- [ ] Probar diagnóstico con muy poca evidencia y porcentajes engañosos.
- [ ] Probar bancos con pocos ejercicios, claves ausentes, duplicados y subejes sin contenido.
- [ ] Probar días omitidos, cambios de dedicación, reinicio de semana y reanudación.
- [ ] Verificar exactamente la proporción **4:3** para 7, 14 y 21 nodos completos, con posiciones intercaladas y azar reproducible.
- [ ] Confirmar 15 respuestas válidas por eje antes de activar el color de dominio; no exigir cubrir todos sus subejes.
- [ ] Unificar primeras respuestas de Aprender, Playground y Ensayos sin duplicar preguntas ya vistas ni correcciones.
- [ ] Comprobar que pasar días sin estudiar un eje **no cambia su dominio**.
- [ ] Validar numéricamente `w_i = 0.95^(k_i/5)` y medir sensibilidad a diferentes parámetros.
- [ ] Comprobar rotación equitativa de 4 ejes en M: 1, 2 o 3 nodos por eje según dedicación.
- [ ] Comprobar elección de 1, 2, 3 o 4 ejes R con dominios muy diferentes o próximos y umbrales de 10/30 puntos.
- [ ] Validar rotación de subejes, comportamiento con un único eje débil, empates y cambio de grupo al mejorar el dominio.
- [ ] Verificar que HLR y la ponderación 70/30 **no participan** en el MVP.

## 8. Dudas y decisiones por cerrar

1. **Diagnóstico inicial:** reconciliar las variantes de 12 y 16 preguntas sin confundirlas con la calibración de 15 respuestas por eje.
2. **Calibración:** decidir si 15 respuestas por eje es un umbral suficiente y cómo mostrar resultados poco representativos sin exigir todos los subejes.
3. **Dominio:** validar el factor `0.95` por cada cinco respuestas nuevas; definir agregación de porcentajes por subeje al eje.
4. **Selección de refuerzos:** validar los umbrales provisionales de 10 y 30 puntos y cómo agrupar ejes/subejes con evidencia insuficiente; decidir empates y recálculo de nodos no iniciados.
5. **Subejes de mantenimiento:** confirmar la rotación equitativa propuesta dentro de cada eje y qué hacer cuando se agotan sus preguntas disponibles. **HLR queda fuera del MVP**.
6. **Intercalado 4:3:** reglas para omisiones, nodos extra, cambios de meta, calendario y replanificación sin alterar lo completado.
7. **Ruta visible:** cuántos nodos futuros se muestran y cómo explicar los cambios sin confundir al estudiante.
8. **Banco:** cómo proceder si no hay cinco preguntas distintas, con clave acreditada y aptas para un subeje.
9. **Corrección:** cuántos reintentos antes de mostrar una resolución guiada y cómo evitar bloqueos.
10. **Evidencia entre modos:** cómo manejar preguntas repetidas, ayudas, tiempos distintos y progreso importado.
11. **XP y rachas:** cómo motivar sin inflar el dominio por reintentos o repeticiones.

Las reglas cuantitativas **no se consideran validadas** hasta probarlas con simulaciones y, posteriormente, estudiantes reales.

## 9. Próximos pasos

- [ ] Cerrar las dudas mínimas del diagnóstico y de cobertura.
- [ ] Implementar **primero el planificador 4:3** y el estimador de dominio como funciones puras, independientes de UI.
- [ ] Simular primero el reparto exacto M/R, rotación de los cuatro ejes M y selección/rotación dinámica de 1-4 ejes R y sus subejes, con umbrales provisionales.
- [ ] Probar calibración de 15 respuestas, dominio ponderado, ausencia de sesgos y recalculación sin cambios retrospectivos.
- [ ] Diseñar la pantalla de nodos a partir de la UI actual.
- [ ] Integrar preguntas elegibles del banco *Default* y el historial unificado de Aprender/Playground/Ensayos, conservando primeros intentos.
- [ ] Probar el recorrido con estudiantes y ajustar la distribución.

### Referencias de proyecto

- Web: [`sebinsebon/paessed.cl`](https://github.com/sebinsebon/paessed.cl), rama de trabajo [`feat/aprender`](https://github.com/sebinsebon/paessed.cl/tree/feat/aprender) (creada el 2026-11-04; sin implementación nueva del modo en el momento de redactar esta nota).
- Documentación vecina: [[sistema/SISTEMA_ADAPTATIVO]], [[sistema/ESTRUCTURA_M1]], [[sistema/BANCO_PREGUNTAS]], [[sistema/ESTADISTICAS]], [[DISENO]].
- Registro histórico de decisiones: [`docs/DECISIONS.md`](../../docs/DECISIONS.md). Puede contener criterios anteriores que deben revisarse expresamente; no asumir que esta nota los modifica.
