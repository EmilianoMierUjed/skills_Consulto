---
name: jurisprudencia
description: "Búsqueda estructurada de jurisprudencia y tesis SCJN vía el conector MCP legal, con priorización de jurisprudencia obligatoria por circuito y links oficiales al Semanario. Usa esta skill cuando el usuario pida busca jurisprudencia, busca tesis, criterios SCJN, precedentes obligatorios, jurisprudencia para un amparo o demanda, fundamenta con jurisprudencia, o variantes similares de litigio mexicano."
---

# Jurisprudencia SCJN — búsqueda con priorización por circuito

Investigar criterios jurisdiccionales mexicanos para litigio usando **las tools del conector MCP legal**
(`list_databases`, `describe_schema`, `sample_rows`, `run_sql`, `fts_help`) sobre la base `jurisprudencias`.
Nunca inventes registros, rubros ni texto: todo dato citado debe venir de `run_sql`.

## 0. Contexto del caso (una sola vez por conversación)

Antes de buscar, determina **circuito** y **materia** del caso:
- Dedúcelos de lo que cuente el abogado (entidad federativa, tribunal, tipo de juicio).
- Si no se deducen, **pregunta una sola vez**: "¿En qué circuito (o estado) litigas este asunto y en qué materia?".
- Si el abogado no lo sabe o no aplica, asume alcance nacional y dilo expresamente.

Región del circuito (para Plenos Regionales, Acuerdo General 67/2022 CJF):
- **Centro-Norte**: circuitos 1º (solo materias penal y administrativa), 2º, 4º, 5º, 8º, 9º, 12º, 15º, 16º, 17º, 19º, 22º, 23º, 24º, 25º, 26º, 28º y 30º.
- **Centro-Sur**: circuitos 1º (solo materias civil y de trabajo), 3º, 6º, 7º, 10º, 11º, 13º, 14º, 18º, 20º, 21º, 27º, 29º, 31º y 32º.

## 1. Búsqueda (FTS5 escalonado, vía run_sql)

Llama `fts_help` al inicio si es tu primera consulta (trae la sintaxis y las reglas de links oficiales).
Busca en fases y detente cuando tengas material suficiente:
1. `rubro:` con términos precisos → 2. frase exacta → 3. combinaciones `AND` → 4. sinónimos jurídicos → 5. `OR` amplio solo como rescate.

Ejemplo base:
```sql
SELECT t.id_tesis, t.rubro, t.tipo_tesis, t.epoca, t.instancia, t.organo_juris,
       t.tesis_codigo, t.anio, t.nota_publica,
       snippet(tesis_fts, 1, '[', ']', '...', 32) AS fragmento
FROM tesis_fts JOIN tesis t ON t.rowid = tesis_fts.rowid
WHERE tesis_fts MATCH 'NEAR(interes superior menor, 6)'
ORDER BY bm25(tesis_fts) LIMIT 25
```
Abre `texto` y `precedentes` solo de los registros realmente útiles (SELECT dirigido por `id_tesis`).

**Filtro de relevancia:** un criterio pasa al análisis solo si conecta con al menos dos ejes del caso
(acto/norma, derecho afectado, materia, efecto procesal, hechos, autoridad). Descarta coincidencias de palabra aislada.

**Trampa al filtrar por circuito:** `organo_juris LIKE '%SÉPTIMO CIRCUITO%'` también atrapa el DÉCIMO
SÉPTIMO y el VIGÉSIMO SÉPTIMO. Usa el patrón con "DEL": `LIKE '%DEL SÉPTIMO CIRCUITO%'` (los ordinales
compuestos nunca llevan "DEL" justo antes del ordinal simple). Verifica siempre `organo_juris` en el
resultado antes de etiquetar un criterio como "del circuito del caso".

## 2. Ranking de obligatoriedad (ORDEN ESTRICTO)

Clasifica cada hallazgo con `tipo_tesis`, `instancia`, `organo_juris` y `epoca`, y ordénalo así:

1. **Jurisprudencia de la SCJN** (Pleno > Salas) — obligatoria para todos los órganos del país.
2. **Jurisprudencia del Pleno Regional de la región del caso** (o Pleno de Circuito del circuito del caso, figura previa a 2023) — obligatoria dentro de su región/circuito.
3. **Jurisprudencia de Tribunales Colegiados del MISMO circuito** — obligatoria dentro del circuito.
4. **Jurisprudencia obligatoria de otro circuito o región, aplicable por analogía** — no vincula en el circuito del caso, pero SIEMPRE pesa más que cualquier tesis aislada. Explica la analogía (mismos supuestos, misma razón jurídica).
5. **Tesis aisladas** (SCJN > TCC) — solo orientadoras; márcalo expresamente.

Reglas de vigencia:
- Prefiere **11ª y 12ª Época**. Criterios de 10ª siguen siendo citables; de 9ª o anteriores, cítalos con reserva expresa ("época anterior, verificar vigencia") y solo si no hay criterio reciente.
- Revisa `nota_publica` y `precedentes`: si el criterio fue **interrumpido, sustituido o superado por contradicción**, dilo y no lo presentes como vigente.
- En 11ª época la jurisprudencia de SCJN también se forma **por precedentes** (sentencias con votación calificada); no descartes un criterio solo por no decir "por reiteración".

## 3. Links oficiales — OBLIGATORIO

Cada criterio citado lleva su link clickeable al Semanario Judicial de la Federación:
`https://sjf2.scjn.gob.mx/detalle/tesis/{id_tesis}` (el `id_tesis` es el registro digital).
Por eso **siempre incluye `id_tesis` en tus SELECT**. No inventes registros; si no tienes `id_tesis`, no hay link ni cita.
(El portal bloquea a bots con 403; el link es para que el abogado lo abra en su navegador.)

## 4. Salida estructurada (siempre)

Formato de ficha por criterio:

```markdown
### [OBLIGATORIA — Séptimo Circuito] {rubro}
- **Registro:** {id_tesis} · [Ver en el Semanario](https://sjf2.scjn.gob.mx/detalle/tesis/{id_tesis})
- **Tesis:** {tesis_codigo} · {tipo_tesis} · {epoca} · {instancia}
- **Órgano:** {organo_juris}
- **Obligatoriedad:** {nacional | obligatoria en el circuito X | aplicable por analogía (obligatoria en el circuito Y) | orientadora} — y por qué
- **Extracto pertinente:** "{cita textual breve del texto}"
- **Aplicación al caso:** {cómo se usa en el litigio concreto}
```

Etiquetas de encabezado: `[OBLIGATORIA — {ámbito}]`, `[OBLIGATORIA POR ANALOGÍA — {circuito de origen}]`, `[ORIENTADORA]`, `[ADVERSA]`.

Documento final (compacto, 600–1,200 palabras; 3–8 criterios):

```markdown
# Jurisprudencia — {tema}
**Caso:** {circuito, materia, problema jurídico}

## 1. Respuesta corta
## 2. Criterios obligatorios aplicables al caso
## 3. Criterios obligatorios aplicables por analogía
## 4. Criterios orientadores (tesis aisladas)
## 5. Criterios adversos o riesgos
## 6. Tabla resumen
| # | Registro | Rubro (corto) | Tipo | Obligatoriedad | Link |
```

- Incluye la sección 5 solo si encontraste criterios adversos útiles para anticipar.
- No imprimas estrategia de búsqueda, descartes ni bitácora **salvo** que el usuario pida
  `exhaustivo`, `anexo`, `bitácora`, `trazabilidad completa`, `auditoría` o `todos los resultados`;
  en ese caso agrega: Estrategia FTS5, Criterios descartados y razón, y Bitácora de consultas SQL.

## 5. Disciplina

- Cada registro se cita individualmente (nada de "registros 2029001-2029010").
- Si la búsqueda no arroja criterios pertinentes, dilo con claridad y sugiere reformulación; no rellenes con criterios marginales.
- Para estándares internacionales (tratados, Corte IDH) usa la skill de corpus iuris; para inaplicar normas, la de control difuso.
