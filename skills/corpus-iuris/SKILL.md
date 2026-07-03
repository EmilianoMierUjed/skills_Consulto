---
name: corpus-iuris
description: "Búsqueda estructurada de tratados, Corte IDH, CIDH, comités ONU, OIT y UHRI vía el conector MCP legal, con links oficiales en cada fuente. Usa esta skill cuando el usuario pida busca tratados, estándares internacionales, Corte IDH, corpus iuris, bloque convencional, comités ONU, CIDH, derecho internacional de los derechos humanos, o variantes similares para litigio mexicano."
---

# Corpus iuris internacional — búsqueda con links oficiales

Investigar normas y estándares internacionales de derechos humanos para litigio mexicano usando
**las tools del conector MCP legal** (`run_sql`, `describe_schema`, `fts_help`) sobre la base `corpus_iuris`.
Nunca inventes tratados, artículos, casos ni órganos: todo dato citado sale de `run_sql`.

## 1. Estrategia de entrada

- Si ya hay nombre de caso, instrumento, símbolo ONU u identificador → búsqueda por metadatos directa en la tabla específica.
- Si el tema es abierto → entra por `metadata_unificada`/`fts_corpus` y baja a la tabla específica vía `fuente` + `fuente_id`.
- FTS5 por fases: frase exacta → `AND` → términos por derecho/estándar → `OR` amplio solo como rescate.
- Itera por tipo de fuente según el caso: `articulos`/`instrumentos`, `corteidh_casos`, `corteidh_opiniones`,
  `cidh_informes`, `observaciones_generales`, `comite_dictamenes`, `relatores_informes`, `uhri_recomendaciones`, `oit_convenios`.
- `derechos_relacionados` es JSON: filtra con las funciones JSON de SQLite.
- **No uses** `cidh_casos` ni `scjn_tesis_convencionalidad`: están vacías en la versión actual de la base.

**Filtro de relevancia:** una fuente pasa al análisis solo si conecta con al menos dos ejes
(derecho afectado, estándar aplicable, hechos, acto estatal, tipo de fuente, efecto litigante).

## 2. Jerarquía de fuentes (para litigio mexicano)

1. **Vinculantes:** tratados ratificados por México (cualquier materia, no solo DDHH) y **jurisprudencia de la Corte IDH**
   (contenciosos, opiniones consultivas, medidas provisionales, supervisión de cumplimiento) — parámetro de regularidad
   conforme a CT 293/2011.
2. **Orientadoras:** CIDH (informes), comités ONU (dictámenes y observaciones generales), relatorías, UHRI,
   convenios OIT no ratificados. Úsalas cuando amplíen la protección y dilo expresamente ("criterio orientador").
- Verifica la pertinencia para México (columna `pais` cuando exista); un dictamen contra otro Estado sirve como estándar
  interpretativo, no como condena aplicable.

## 3. Links oficiales — OBLIGATORIO

Cada fuente citada lleva su link oficial clickeable. **Siempre incluye las columnas de URL en tus SELECT:**

| Tabla | Columna(s) de link |
|---|---|
| `instrumentos` | `url_texto` (a veces NULL) |
| `corteidh_casos` | `url_pdf` (PDF oficial corteidh.or.cr) |
| `corteidh_opiniones` | `url_pdf` |
| `cidh_informes` | `url_pdf` (preferente), `url_html` |
| `oit_convenios` | `url_normlex` |
| `comite_dictamenes` | `url` |
| `observaciones_generales` | `url` |
| `relatores_informes` | `url` |
| `uhri_recomendaciones` | sin columna: construye `https://undocs.org/es/{simbolo_documento}` |

Si la URL viene NULL, dilo y sugiere el portal oficial de búsqueda (OEA, corteidh.or.cr, NORMLEX, ohchr.org);
**nunca inventes un link**.

## 4. Salida estructurada (siempre)

Ficha por fuente:

```markdown
### [VINCULANTE] {título / caso / artículo}
- **Fuente:** {Corte IDH | Tratado | Comité X | CIDH | OIT | Relatoría | UHRI} · {identificador: serie C No., símbolo ONU, No. de convenio}
- **Link oficial:** [{texto descriptivo}]({url})
- **Estándar:** {qué establece, en 1-3 frases con cita textual breve}
- **Aplicación al caso:** {cómo se invoca en el litigio}
```

Etiquetas: `[VINCULANTE]` u `[ORIENTADORA]`.

Documento final (compacto, 600–1,200 palabras; 4–8 fuentes centrales):

```markdown
# Corpus iuris — {tema}
## 1. Respuesta corta
## 2. Fuentes vinculantes (tratados y Corte IDH)
## 3. Fuentes orientadoras (CIDH, comités, relatorías, UHRI, OIT)
## 4. Uso litigante sugerido
## 5. Tabla resumen
| # | Fuente | Identificador | Carácter | Link |
```

- Agrupa artículos de un mismo instrumento cuando sostengan el mismo punto; nada enciclopédico.
- Modo exhaustivo (estrategia, descartes, bitácora) solo si el usuario pide `exhaustivo`, `anexo`, `bitácora`,
  `trazabilidad completa`, `auditoría` o `todos los resultados`.
- Para cruzar con jurisprudencia nacional usa la skill de jurisprudencia; para inaplicar una norma, la de control difuso.
