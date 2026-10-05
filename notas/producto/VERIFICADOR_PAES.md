# Verificador PAES — producto web

Estado: especificación de producto; implementación pendiente. Este Verificador
muestra ensayos y resultados del estudiante. La **revisión de fidelidad de
Etapa 2** es un proceso técnico diferente, explicado en
[Pipeline del importador](../../docs/IMPORTADOR.md). El formato objetivo de importación es `final.json`;
su contrato todavía no está implementado. Importar un ensayo privado no lo
publica en el banco.

## Proposito

Verificador PAES permite analizar ensayos que el estudiante ya realizo, usando un archivo estructurado importado en la web.

## Experiencia del usuario

La web recibira posteriormente contenido verificado. Durante el desarrollo de la Etapa 1, el artefacto intermedio se llama `draft.json` y puede contener incidencias o bloques `unresolved`.

La web deberia mostrar:

- preguntas;
- respuestas del estudiante;
- respuestas correctas;
- correctas;
- incorrectas;
- omitidas;
- explicacion;
- eje;
- tema;
- imagenes o graficos si existen.

La revision podria mostrarse como una grilla de preguntas y una vista detallada por pregunta.

## Funcionamiento actual

El [contrato draft v1](../../docs/DRAFT_V1_CONTRACT.md) y su productor están implementados. El [pipeline del importador](../../docs/IMPORTADOR.md) describe sus límites. Respuestas, explicaciones, taxonomía, estadísticas e importación web pertenecen a etapas posteriores; no forman parte del borrador.

Antes de confirmar una importacion, el usuario podra corregir respuestas, metadatos o contenido cuando el Scanner haya leido algo de forma incorrecta o una pregunta sea defectuosa. Solo los resultados confirmados recalibraran el perfil de dominio y las recomendaciones.

Las correcciones del Verificador no otorgaran puntos en el futuro modo Ranking. Ranking usara solamente preguntas diarias controladas por el sistema.

La condición de contenido oficial no sustituye la revisión de permisos. La admisión pública exige fidelidad, revisión educativa y derechos; importar para uso privado es un flujo diferente.

Para ensayos externos, el enfoque sera local-first: el servidor de la plataforma no deberia almacenar PDFs privados originales.

## Preguntas abiertas

- Como se ingresaran las respuestas del estudiante?
- Que validaciones debe hacer la web al importar contenido verificado?
- Como se mostraran y conservaran las correcciones antes de confirmar el ensayo?
- Como se revierte una confirmacion incorrecta sin perder trazabilidad?
- Como manejar imagenes muy pesadas dentro del JSON?

## Modos de procesamiento por evaluar

El flujo puede variar según dónde se ejecute y si el usuario entrega un
solucionario. Esta matriz es una propuesta para la skill; todavía no describe
funcionalidad implementada en la web.

| Entorno | Solucionario | Flujo propuesto |
| --- | --- | --- |
| Chat | Sí | Importación directa con el solucionario como fuente de respuestas. |
| CLI | Sí | Importación directa o flujo avanzado, según la capacidad del CLI para leer el PDF y sus visuales. |
| CLI | No | Procesamiento completo; puede requerir agentes para repartir extracción y resolución. |
| Chat | No | Experimental; primero hay que comprobar lectura del PDF, visuales y contexto disponible. |

La presencia de un solucionario no demuestra por sí sola que corresponda al
ensayo ni que sus respuestas sean correctas. Se debe registrar su fuente y
mantener visibles las respuestas pendientes de comprobación.

## Comparación experimental local

En el laboratorio Test2 se prepararon dos variantes AGY para comparar un mismo
PDF mediante observación manual:

- **V1:** producción inicial desde el PDF con la skill PDF-directo, seguida por
  una auditoría independiente por lotes y aprobación explícita de correcciones.
- **V2:** localización de preguntas y generación directa desde fragmentos del
  PDF por agentes independientes, sin usar el resultado semántico de V1.

Son experimentos locales de importación y no una implementación del Verificador
web ni una medición de precisión oficial. Tiempo, contexto, uso, comportamiento
de agentes y calidad se comparan manualmente. Cada variante usa la misma fuente
PDF, pero conserva sus propios artefactos y estados de respuesta.

## Respaldo y publicación

El laboratorio y sus corridas permanecen en `Test2/Final`. No se deben agregar
al repositorio público los PDFs de ensayo ni los resultados, imágenes o demás
artefactos derivados. Esto sigue la [política de datos y contenido](../../docs/DATA_AND_CONTENT_POLICY.md).
Si se necesita respaldo de las Skills, considerar un repositorio privado
separado y excluir los PDFs y outputs; revisar licencias y contenido antes de
publicar cualquier dependencia.

## Pendientes

- [ ] Definir visualización de grilla.
- [ ] Definir vista por pregunta.
- [ ] Definir importación de contenido verificado (`final.json`).
- [ ] Definir relación con estadísticas.
- [ ] Definir límites de privacidad y almacenamiento.
- [ ] Evaluar la matriz de modos con ensayos y solucionarios autorizados.
