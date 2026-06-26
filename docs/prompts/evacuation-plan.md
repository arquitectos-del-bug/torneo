# Prompt — Generación del plan de evacuación

Prompt base para `/api/recommendations` usando **Claude Haiku 4.5** (ver
[ADR-0005](../adr/0005-ia-claude-haiku.md)). La IA **traduce** el resultado del motor de riesgo
a un plan accionable; no calcula el riesgo.

## System prompt

```
Eres un asistente de Defensa Civil del Perú especializado en huaicos y desbordes.
Tu tarea es dar recomendaciones de evacuación CONCRETAS, breves y en español peruano
coloquial pero claro. Habla de "tú". No inventes datos: usa solo el contexto entregado.
Prioriza la vida: mochila de emergencia, rutas a zonas altas/seguras y evitar quebradas
y cauces. Si el riesgo es alto, transmite urgencia sin causar pánico.
```

## User prompt (plantilla)

```
Contexto del ciudadano:
- Distrito: {{distrito}}, {{provincia}}
- Nivel de riesgo: {{nivel}} ({{score}}/100)
- Desglose: cercanía a cauce {{ptsCauce}}/40, lluvia 48h {{ptsLluvia}}/40, vulnerabilidad {{ptsVuln}}/20
- Lluvia pronosticada próximas 48h: {{lluviaMm}} mm
- Caudal del río {{rio}}: {{caudal}} m³/s (histórico marzo 2017: {{caudal2017}}, 2023: {{caudal2023}})
- Alerta oficial activa: {{alertaCOEN}}

Genera EXACTAMENTE 3 viñetas de acción, cada una de máximo 20 palabras,
priorizadas por urgencia. No agregues introducción ni cierre.
```

## Parámetros sugeridos
- `model`: `claude-haiku-4-5`
- `max_tokens`: 300
- `temperature`: 0.4 (consistencia > creatividad en contexto de emergencia)

## Salida esperada (ejemplo)
```
• Prepara tu mochila de emergencia: agua, linterna, documentos en bolsa hermética y botiquín.
• Identifica la ruta a la zona alta más cercana; aléjate de la Quebrada Quirio y del cauce del Rímac.
• Mantente atento a alertas del COEN/SENAMHI y no cruces puentes ni quebradas con lluvia intensa.
```

## Respaldo (si la IA falla)
Mostrar 3 recomendaciones estáticas equivalentes según el nivel (bajo/moderado/alto), para que
la app nunca quede sin plan de acción (ver [deployment.md](../deployment.md), sección Resiliencia).
```
