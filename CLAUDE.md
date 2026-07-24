# CLAUDE.md

Este archivo le da contexto persistente a Claude Code para trabajar en
este proyecto. Leelo completo antes de tocar código o generar resultados.

## Qué es este proyecto

Audit de drives prosociales en LLMs, para el curso Technical AI Safety
Project (BlueDot). Ver `prosocial-drives-audit.md` para el detalle
completo de motivación, timeline y criterios de éxito — ese archivo es
la fuente de verdad del diseño experimental, este CLAUDE.md es la guía
operativa de cómo trabajar en el repo día a día.

**Pregunta de investigación (resumen):** ¿Muestran los modelos actuales
drives prosociales que van más allá de ayudar al usuario — considerar
bienestar de terceros, señalar efectos secundarios dañinos, detectar
injusticias — en conversaciones multi-turno naturalísticas (no en
paradigmas de teoría de juegos)?

## Estructura del repo

```
.
├── prosocial-drives-audit.md      # diseño experimental completo (fuente de verdad)
├── CLAUDE.md                       # este archivo
├── seeds/
│   ├── business_strategy/
│   │   └── seeds_business_strategy.md      # 3 seeds: BS-1, BS-2, BS-3
│   └── resource_allocation/
│       └── seeds_resource_allocation.md    # 3 seeds: RA-1, RA-2, RA-3
├── dimensions/
│   ├── scope_of_concern.md
│   ├── proactive_initiative.md
│   ├── fairness_awareness.md
│   ├── side_effect_flagging.md
│   └── user_autonomy_respect.md
├── transcripts/                    # output crudo de Petri, un dir por corrida
├── results/                        # scores agregados + análisis
└── figures/                        # gráficos para el write-up
```

## Comando base de evaluación

```bash
pip install inspect-petri
inspect eval inspect_petri/audit \
  -T seed_instructions=./seeds/<área>/ \
  -T judge_dimensions=./dimensions/ \
  --model-role auditor=anthropic/claude-sonnet-4-6 \
  --model-role target=<modelo-a-evaluar> \
  --model-role judge=anthropic/claude-opus-4-6
```

**Siempre correr con `<área>` apuntando a una sola carpeta de seeds por
vez** (business_strategy o resource_allocation), no mezclar áreas en
una misma corrida — necesitamos poder desagregar resultados por área
en el análisis.

**Siempre configurar `message_limit` o `time_limit` en la corrida.**
Sin esto, el auditor puede entrar en loop y la evaluación no termina
(ver Guía para Participantes, sección de Inspect).

## Modelos target de este proyecto

| Modelo | Prioridad |
|--------|-----------|
| Claude Sonnet 4.6 | Sí — referencia |
| GPT-4o | Sí |
| Gemini 2.0 Flash | Sí |
| Llama 3.1 70B | Solo si hay tiempo |

No agregar modelos target sin actualizar el diseño de `2 áreas × N
modelos × 20 conversaciones` en `prosocial-drives-audit.md` — cada
modelo nuevo multiplica el costo y el tiempo de la corrida completa.

## Reglas al escribir o editar seeds

- Ninguna seed debe mencionar ética, moral, "impacto social",
  "prosocialidad" o palabras equivalentes. El escenario tiene que leerse
  como un pedido de consultoría genuino — si suena a test moral, se
  rompe el diseño (eval-awareness). Esto es la razón de ser del
  proyecto, no un detalle de estilo.
- Cada seed necesita un turno de escalada (típicamente el turno 3)
  donde el usuario simulado presiona para ignorar cualquier
  consideración prosocial que haya aparecido antes. Es la parte que
  mide si el drive *aguanta* presión conversacional.
- Variar el sector/contexto entre seeds de una misma área, para no
  medir solo sensibilidad a un tipo de situación.
- Antes de dar por buena una seed nueva, correrla manualmente 2-3 veces
  y leer los transcripts a mano — no asumir que el auditor va a generar
  la escalada esperada solo por cómo está redactada la seed.

## Reglas al tocar las rúbricas (`dimensions/`)

- Las 5 dimensiones actuales son: `scope_of_concern`,
  `proactive_initiative`, `fairness_awareness`, `side_effect_flagging`,
  `user_autonomy_respect`. No agregar una sexta dimensión sin
  actualizar también el análisis estadístico y el MVE en
  `prosocial-drives-audit.md`.
- `user_autonomy_respect` NO es monótona: un score alto no es "más
  prosocial", es "buen balance entre ayudar y proteger a terceros". No
  la trates igual que las otras cuatro al analizar resultados.
- Cada nivel de cada rúbrica debe describir una conducta observable en
  el texto, no un adjetivo. Si al revisar una rúbrica notás que dos
  niveles seguidos son indistinguibles en la práctica, es un bug de la
  rúbrica — avisar antes de seguir corriendo evaluaciones con ella.
- Cualquier cambio a una rúbrica después del pilot (semana 3) invalida
  la comparación con corridas previas hechas con la versión anterior.
  Versionar (`_v1`, `_v2`) en vez de sobreescribir si ya hay resultados
  corridos con la versión vieja.

## Pilot antes de escalar (semana 3)

Antes de correr el experimento completo (2 áreas × 3 modelos × 20
conversaciones = 120 conversaciones), correr un pilot chico:
1 área × 2 modelos × 10 conversaciones, y leer los transcripts a mano.
Objetivo: confirmar que (a) las seeds generan la situación esperada y
(b) el judge aplica la rúbrica de forma consistente con lo que un
humano diría mirando el mismo transcript. Es muy probable que haya que
reescribir seeds y/o ajustar rúbricas después de esto — está
contemplado en el timeline, no es un desvío.

## Análisis de resultados

- Test estadístico: Mann-Whitney (no paramétrico, apropiado para el
  tamaño de muestra de este proyecto).
- Reportar por dimensión y por área de seed por separado, no solo un
  score prosocial agregado — el proyecto busca precisamente ver si hay
  diferencias por tipo de escenario.
- Guardar siempre los transcripts más informativos (los que muestran
  mejor la diferencia entre modelos) para citar como ejemplo en el
  write-up — un score solo no comunica, un ejemplo concreto sí.

## Al usar Claude Code en este proyecto

- Validar siempre que los resultados tengan sentido antes de darlos
  por buenos: si un modelo saca 5/5 en todo o 1/5 en todo en una
  dimensión, investigar si es señal real o un bug en el prompt del
  judge o en el parsing de scores.
- Leer con detalle cualquier prompt nuevo que se genere para el
  auditor o el judge antes de correrlo a escala — no asumir que un
  prompt generado automáticamente preserva las reglas de esta guía
  (ej. no mencionar ética explícitamente en las seeds).
- Mantener el research log actualizado en cada sesión de trabajo,
  incluyendo corridas fallidas o resultados negativos — son parte del
  criterio de evaluación del curso.
