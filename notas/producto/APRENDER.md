---
title: Aprender — ruta de nodos adaptativos
tags:
  - paessed
  - producto
  - aprender
  - ui-ux
actualizado: 2026-11-04
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
| Personalización | Más presencia de subejes débiles, con mantenimiento de los otros ejes. |
| Proporción semanal | **4 nodos de mantenimiento por cada 3 de refuerzo**, intercalados. |
| Ubicación de refuerzos | Posiciones con aleatoriedad controlada; cuotas exactas y ruta guardada. |
| Dominio | Respuestas originales válidas, procedentes de Aprender, Playground y Ensayos; ponderación por nuevas respuestas del mismo subeje, no por días. |
| Calibración | **15 respuestas válidas por eje** para dejar el estado gris provisional; sin cobertura forzosa de cada subeje. |
| Repaso | HLR como propuesta separada, pendiente de validar para matemáticas. |
| Frecuencia | Nodos futuros recalculables conforme se completa práctica. |
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

## 3. Planificador semanal: proporción 4:3

**Dirección conversada:** por cada **7 nodos**, programar **4 de mantenimiento (M)** y **3 de refuerzo (R)**. Mantener la proporción para las tres preferencias de dedicación y **mezclar ambos tipos durante la semana**, sin reservar días completos a uno solo.

| Preferencia | Total en siete días completos | Mantenimiento | Refuerzo |
| --- | ---: | ---: | ---: |
| Poco tiempo (1/día) | 7 | 4 | 3 |
| Normal (2/día) | 14 | 8 | 6 |
| Más tiempo (3/día) | 21 | 12 | 9 |

- **Mantenimiento:** rotación equilibrada de los cuatro ejes; dentro del eje elegir un subeje adecuado sin imponer desbloqueos.
- **Refuerzo:** seleccionar los subejes que necesitan más práctica mediante los resultados recientes, errores y repasos pendientes. La fórmula exacta de prioridad aún no está aprobada.
- **Posiciones:** **aleatoriedad controlada** para intercalar M y R; **las cantidades totales no quedan al azar**.
- **Ruta estable:** guardar la distribución generada para que recargar no cambie los nodos ya visibles. Los completados nunca se reescriben.
- **Adaptación:** al terminar cada nodo se pueden reajustar los subejes de los nodos futuros, manteniendo las cuotas pendientes y el historial.

### Ejemplo de dispersión controlada

En la modalidad de 3 nodos por día, para alcanzar **9 R y 12 M** en siete jornadas sin hacer días exclusivos:

- Cinco días: **M, M y R** en algún orden.
- Dos días: **M, R y R** en algún orden.
- Elegir de forma reproducible qué dos días llevan dos refuerzos y mezclar las posiciones dentro del día.

Para 2 nodos diarios, seis días llevarían un M y un R, y uno llevaría dos M. Para 1 nodo diario, habría cuatro jornadas M y tres R, con las posiciones intercaladas.

Estas son **metas programadas, no resultados realizados**: si el estudiante omite días, no marcar nodos como completados. Queda por diseñar cómo replanificar sesiones perdidas, cambios de dedicación y práctica extra.

### Evitar monotonía y falta de cobertura

- Los cuatro ejes deben aparecer en mantenimiento; con 3 nodos diarios, los 12 M pueden repartirse en **3 por eje**.
- En refuerzo debe haber prioridad de debilidades, pero no repetir innecesariamente el mismo subeje en nodos consecutivos.
- El algoritmo trabaja **por subeje** (16), aunque los nodos muestren también su eje.
- Una posición R **no significa elegir un eje por sorteo ponderado**. Esa idea se descartó por poder producir semanas injustamente concentradas.
- La garantía de cuatro ejes no obliga a cubrir los 16 subejes todas las semanas; estos necesitan seguimiento gradual.

### Ejemplo ilustrativo de distribución por eje

Para un estudiante con Geometría y Probabilidad débiles que complete 14 nodos:

| Eje | Nodos de ejemplo |
| --- | ---: |
| Geometría | 6 |
| Probabilidad y Estadística | 4 |
| Álgebra y Funciones | 2 |
| Números | 2 |
| **Total** | **14** |

No son porcentajes obligatorios: el reparto final depende de los subejes, la evidencia y el contenido disponible.

## 4. Calibración, dominio y HLR

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

**Cambio importante respecto de ideas anteriores:** no reducir automáticamente el dominio por no entrar a la web ni por dejar un eje sin practicar. Los días transcurridos deben influir en el **repaso HLR**, no castigar porcentajes de dominio.

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

### HLR: repaso espaciado separado del dominio

La fórmula de referencia publicada por Duolingo para la probabilidad de recordar un contenido es:

~~~math
P(recordar) = 2^(-t/h)
~~~

- `t` = tiempo desde el último repaso.
- `h` = semivida de memoria estimada con el historial.
- HLR ayuda a decidir **cuándo sería conveniente repasar**, incluso si el dominio histórico es alto.
- Una alerta de HLR **no disminuye automáticamente el dominio**. Solo una nueva respuesta elegible puede modificarlo.
- Ejemplo: Porcentajes conserva 85 % de dominio, pero lleva tiempo sin practicar; el planificador puede incluir un repaso. El dominio cambia únicamente según el resultado posterior.

**Precaución:** HLR fue desarrollado sobre aprendizaje de palabras/conceptos lingüísticos y requiere parámetros estimados con datos. No está validado automáticamente para ejercicios matemáticos por subeje ni tenemos todavía un modelo entrenado para PAESSED. Por ahora es una **referencia de diseño experimental**, no un componente implementado ni una probabilidad fiable de resolver preguntas M1.

Referencia técnica: [Duolingo — Half-Life Regression](https://research.duolingo.com/papers/settles.acl16.pdf).

### Separación de responsabilidades

| Componente | Qué decide | Qué no debe hacer |
| --- | --- | --- |
| **Planificador 4:3** | Posiciones de mantenimiento/refuerzo y metas semanales. | Cambiar el dominio sin respuestas. |
| **Estimador de dominio** | Porcentaje observado ponderado por nuevas respuestas del subeje. | Penalizar solo por paso de días. |
| **HLR / programación de repasos** | Cuándo conviene volver a practicar un conocimiento. | Reemplazar resultados originales o declarar dominio verdadero. |
| **Selector de subejes** | Qué contenido entra en cada M o R según las señales disponibles. | Elegir siempre el peor o depender solo de sorteos. |

Todos estos sistemas deben ser verificables con simulaciones y explicables en la interfaz.

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
- Señalar «Refuerzo» o «Repaso» cuando corresponda, pero sin tratarlo como una dificultad de la pregunta.
- Mostrar progreso real por ejes y una explicación corta de por qué se recomienda el nodo (p. ej., «Semejanza: errores recientes»).
- Mantener la identidad visual de PAESSED y reutilizar el visor de preguntas a pantalla completa de Ensayos/Playground cuando sea viable.
- Preparar escritorio, móvil, teclado, accesibilidad y movimiento reducido.

### Ruta dinámica

- Los nodos **completados** permanecen en el historial.
- Los nodos **futuros** son previsiones, susceptibles de reordenarse.
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
- [ ] Evaluar HLR separadamente como recordatorio de repaso, sin alterar automáticamente el dominio.

## 8. Dudas y decisiones por cerrar

1. **Diagnóstico inicial:** reconciliar las variantes de 12 y 16 preguntas sin confundirlas con la calibración de 15 respuestas por eje.
2. **Calibración:** decidir si 15 respuestas por eje es un umbral suficiente y cómo mostrar resultados poco representativos sin exigir todos los subejes.
3. **Dominio:** validar el factor `0.95` por cada cinco respuestas nuevas; definir agregación de porcentajes por subeje al eje.
4. **Selección de refuerzos:** definir la prioridad que combine debilidad medida, errores recientes y revisión pendiente, evitando monotonía.
5. **HLR:** decidir si se utiliza en el MVP, con qué parámetros iniciales y cómo se valida para matemática M1; por ahora es propuesta experimental.
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
- [ ] Construir simuladores y tests para probar cuotas, intercalado controlado, calibración de 15 respuestas, prioridad de subejes y ausencia de sesgos.
- [ ] Diseñar la pantalla de nodos a partir de la UI actual.
- [ ] Integrar preguntas elegibles del banco *Default* y el historial unificado de Aprender/Playground/Ensayos, conservando primeros intentos.
- [ ] Probar el recorrido con estudiantes y ajustar la distribución.

### Referencias de proyecto

- Web: [`sebinsebon/paessed.cl`](https://github.com/sebinsebon/paessed.cl), rama de trabajo [`feat/aprender`](https://github.com/sebinsebon/paessed.cl/tree/feat/aprender) (creada el 2026-11-04; sin implementación nueva del modo en el momento de redactar esta nota).
- Documentación vecina: [[sistema/SISTEMA_ADAPTATIVO]], [[sistema/ESTRUCTURA_M1]], [[sistema/BANCO_PREGUNTAS]], [[sistema/ESTADISTICAS]], [[DISENO]].
- Registro histórico de decisiones: [`docs/DECISIONS.md`](../../docs/DECISIONS.md). Puede contener criterios anteriores que deben revisarse expresamente; no asumir que esta nota los modifica.
