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

## 3. Algoritmo combinado (propuesta, no fórmula aprobada)

### Qué debe equilibrar

1. **Debilidad:** dar prioridad a subejes con resultados bajos, considerando el historial.
2. **Repaso:** reintroducir subejes con errores anteriores o revisión pendiente.
3. **Variedad:** rotar contenidos y evitar sesiones excesivamente repetitivas.
4. **Cobertura obligatoria:** practicar cada uno de los cuatro ejes al menos una vez por semana, incluso con un solo nodo diario.

### NO seleccionar el eje por sorteo

Se descartó la idea de hacer una selección aleatoria ponderada de ejes. Aunque Geometría tenga prioridad alta, un sorteo podría producir una semana dominada por Geometría y excluir otros ejes.

**Alternativa preferida: planificador determinista por cuotas.**

1. Calcular cuántos nodos están disponibles para la semana según la preferencia: 7, 14 o 21 si se estudia cada día.
2. Reservar al menos **un nodo por eje** para garantizar cobertura.
3. Distribuir los nodos restantes mediante puntuaciones de prioridad de debilidad, repaso e intervalo sin practicar, sujetas a reglas de diversidad.
4. Dentro de cada eje, escoger el **subeje** más conveniente según evidencia y disponibilidad de preguntas.
5. Ordenar los nodos para no encadenar innecesariamente el mismo subeje.
6. Tras cada nodo completado, actualizar el perfil y **replanificar únicamente los nodos futuros**, respetando la cobertura pendiente.

Los cálculos deben producir la misma recomendación a partir del mismo estado e historial. No confundir el orden determinista con una estimación infalible de dominio.

### Ejemplo ilustrativo

Para un perfil ficticio con Geometría 30 %, Probabilidad y Estadística 45 %, Álgebra y Funciones 75 % y Números 90 %, un estudiante de **2 nodos por día** podría recibir:

| Eje | Nodos semanales | Preguntas |
| --- | ---: | ---: |
| Geometría | 6 | 30 |
| Probabilidad y Estadística | 4 | 20 |
| Álgebra y Funciones | 2 | 10 |
| Números | 2 | 10 |
| **Total** | **14** | **70** |

Esto **no es una proporción obligatoria**. Muestra que los dos ejes más débiles reciben más práctica, pero los cuatro permanecen presentes. Los nodos de Geometría pueden alternar subejes distintos, como Semejanza y Transformaciones.

### Regla de cobertura por volumen

| Modalidad | Total semanal máximo previsto | Mínimo reservado a cobertura | Nodos restantes, a distribuir adaptativamente |
| --- | ---: | ---: | ---: |
| 1 al día | 7 | 4 | 3 |
| 2 al día | 14 | 4 | 10 |
| 3 al día | 21 | 4 | 17 |

Son cifras para **siete días completados**. No asignar automáticamente nodos como realizados cuando el estudiante falta. Definir el tratamiento de semanas incompletas y cambios de meta.

**Importante:** un nodo de cada eje por semana no garantiza cubrir sus **16 subejes**. Se necesita además una regla de cobertura gradual por subeje durante varias semanas; su ventana y sus excepciones están pendientes.

### Parámetros pendientes de calibración

- Fórmula de puntuación exacta y pesos de debilidad, repaso y variedad. Se exploró 60/25/15, **sin aprobarla**.
- Cuánto castigar repeticiones recientes del mismo subeje.
- Cuánto historial hace falta para estimar dominio confiable.
- Si se establecen límites máximos de concentración por eje.
- Cómo reaccionar ante mejoras o retrocesos sin cambiar drásticamente toda la ruta.

## 4. Diagnóstico y medición responsable

El diagnóstico inicial sirve para orientar, **no para declarar dominio definitivo**. Una sola respuesta correcta o incorrecta en un subeje es evidencia insuficiente para afirmar que lo domina o que es crítico.

- Estados visuales existentes: **Crítico, Débil, Regular, Fuerte, Dominado**.
- Estado auxiliar: **Sin suficiente evidencia**.
- Distinguir porcentaje de aciertos observado de estimación de dominio.
- Considerar historial, respuestas recientes, omisiones y preguntas ya vistas.
- Una práctica de 5 preguntas modifica el perfil de forma gradual; no convierte automáticamente un subeje en «Dominado».

**Pendiente documental:** `docs/DECISIONS.md` y notas históricas registran un diagnóstico de **12 preguntas**; en conversaciones de octubre se propuso uno de **16 preguntas, una por subeje**. Antes de implementar se debe reconciliar esa diferencia y verificar cobertura y fiabilidad. Esta nota no cambia por sí sola una decisión registrada en `DECISIONS.md`.

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

## 8. Dudas y decisiones por cerrar

1. **Diagnóstico:** ¿12 o 16 preguntas? ¿Cómo se evita una inferencia demasiado fuerte por una sola pregunta de un subeje?
2. **Ventana de cobertura:** ¿qué plazo garantiza visitar todos los subejes sin perjudicar el refuerzo de los más débiles?
3. **Prioridades:** ¿qué fórmula y reglas de diversidad funcionan mejor en las simulaciones?
4. **Ruta visible:** ¿mostrar el plan completo de la semana, unos pocos nodos futuros o solo el próximo?
5. **Dedicación:** ¿se permite completar nodos extra? ¿Se puede cambiar la meta en cualquier momento?
6. **Sesiones perdidas:** ¿reprogramar pendientes o empezar un nuevo plan?
7. **Banco:** ¿qué se muestra si faltan cinco preguntas aptas en un subeje?
8. **Evaluación:** ¿cuándo se muestra la respuesta correcta y cómo influyen intentos repetidos en dominio?
9. **Fuentes de evidencia:** ¿cómo se combinan resultados de Aprender, Ensayos y Playground sin doble conteo?
10. **XP y rachas:** ¿cómo premiar la constancia sin sesgar las estadísticas reales?

## 9. Próximos pasos

- [ ] Cerrar las dudas mínimas del diagnóstico y de cobertura.
- [ ] Implementar **primero el planificador como función pura**, independiente de UI.
- [ ] Construir simuladores y tests para probar cuotas, priorización, cambios y ausencia de sesgos.
- [ ] Diseñar la pantalla de nodos a partir de la UI actual.
- [ ] Integrar preguntas elegibles del banco *Default* y el historial persistente.
- [ ] Probar el recorrido con estudiantes y ajustar la distribución.

### Referencias de proyecto

- Web: [`sebinsebon/paessed.cl`](https://github.com/sebinsebon/paessed.cl), rama de trabajo [`feat/aprender`](https://github.com/sebinsebon/paessed.cl/tree/feat/aprender) (creada el 2026-11-04; sin implementación nueva del modo en el momento de redactar esta nota).
- Documentación vecina: [[sistema/SISTEMA_ADAPTATIVO]], [[sistema/ESTRUCTURA_M1]], [[sistema/BANCO_PREGUNTAS]], [[sistema/ESTADISTICAS]], [[DISENO]].
- Registro histórico de decisiones: [`docs/DECISIONS.md`](../../docs/DECISIONS.md). Puede contener criterios anteriores que deben revisarse expresamente; no asumir que esta nota los modifica.
