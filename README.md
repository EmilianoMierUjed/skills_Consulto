# Consulto — método de amparo y conector a bases jurídicas mexicanas

**Consulto** se compone de dos piezas con licencias y precios distintos a propósito:

1. **El método** (`skills/amparo/`) — flujo de trabajo para litigio de amparo
   mexicano. **Abierto (Apache-2.0) y gratis.** Funciona con o sin el conector.
2. **El conector** — acceso de solo lectura a cinco bases jurídicas mexicanas vía
   MCP. **$150 MXN/mes**, con 20 búsquedas de prueba sin tarjeta.

Regla de diseño de la que cuelga todo:

> **Lo mecánico es de la IA. Lo interpretativo es del humano.**

La IA localiza, coteja y verifica. Nunca interpreta, nunca prioriza, nunca
argumenta. Un amparo es demasiado importante para delegarlo: el abogado que no
construyó el argumento no puede defenderlo en audiencia.

## El método (gratis, en este repo)

Un flujo con fases, checkpoints obligatorios y una **compuerta de fuentes** que
verifica cinco ejes por cada cita: existencia y procedencia, vigencia, ámbito de
obligatoriedad (art. 217 LA), correspondencia textual literal y pertinencia (que
nunca se evalúa: es el acto interpretativo central del litigio).

Modos activos:

- **`fundamentar`** — ya sabes qué pedir, te falta con qué fundarlo: rutas de
  ataque, recolección con ronda adversa obligatoria, compuerta y andamio del
  concepto de violación con los pasos interpretativos vacíos.
- **`viabilidad`** — triage antes de que exista demanda, cuando no sabes si
  tienes caso: acto reclamado, autoridad, interés, definitividad, procedencia
  (art. 61 LA) y la regla de plazo aplicable, sin calcular fechas.

El método corre **sin el conector**: declara `VIGENCIA_NO_VERIFICADA` y
`AMBITO_NO_DETERMINADO` en vez de fingir que verificó, y nunca relaja la
compuerta. El modo sin conector no es una versión limitada: es un método que
dice, en cada corrida, exactamente qué pudo verificar y qué no.

El método completo vive en [`skills/amparo/README.md`](skills/amparo/README.md).

## El conector (de paga)

Las bases que permiten resolver mecánicamente vigencia y obligatoriedad:

| Base | Qué contiene |
|---|---|
| `scjn_tesis.db` | 311,738 tesis y jurisprudencia de la SCJN |
| `scjn_sentencias.db` | 105,856 sentencias con el engrose íntegro |
| `corpus_iuris.db` | 277,239 registros de corpus internacional (Corte IDH, CIDH, comités ONU, OIT) |
| `leyes_federales.db` | 316 leyes federales vigentes, segmentadas en 55,224 artículos |
| `grafo.db` | Grafo de vigencia y obligatoriedad: 472,063 criterios y 298,963 relaciones |

Endpoint: `https://mcp.consulto.page/mcp` · Registro y prueba: <https://consulto.page>

## Instalación

### Claude (app web, escritorio y móvil)

1. Configuración → **Plugins** → Agregar → **Agregar desde un repositorio**.
2. Pega `EmilianoMierUjed/skills_Consulto` y confirma.
3. Instala **Consulto**. La primera vez, Claude abre el inicio de sesión.

### Claude Code

```
/plugin marketplace add EmilianoMierUjed/skills_Consulto
/plugin install consulto-legal-mcp@consulto
```

La primera vez ejecuta `/mcp` para completar el inicio de sesión en el navegador.
Solo el conector, sin el plugin:

```
claude mcp add --transport http consulto https://mcp.consulto.page/mcp
```

### Otros clientes MCP (Cursor, Codex, opencode, ChatGPT…)

Pega el endpoint en la configuración MCP de tu herramienta; el inicio de sesión
OAuth se abre solo. La carpeta `skills/amparo/` sirve como metodología en
cualquier asistente.

Guía paso a paso por herramienta: https://consulto.page/conectar

## Estructura

```
.claude-plugin/   plugin.json + marketplace.json (instalación como plugin)
.mcp.json         servidor MCP con OAuth
skills/amparo/    El método: SKILL.md, MODOS.md, agents/, references/, templates/
```

## Licencia

Apache-2.0. El método se publica para que se adopte, se critique y se mejore.
