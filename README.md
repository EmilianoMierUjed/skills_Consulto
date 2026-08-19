# Consulto — plugin y skills jurídicas

**Consulto** conecta tu asistente de IA (Claude, Claude Code, Cursor, Codex, opencode,
ChatGPT) con jurisprudencia y sentencias de la SCJN (311,673 tesis y 105,803 engroses),
las 316 leyes federales vigentes (55,224 artículos) y el corpus internacional de derechos
humanos (277,239 documentos: Corte IDH, CIDH, comités ONU, OIT, tratados). Cada respuesta
trae su cita con registro digital, documento oficial o link verificable.

No necesitas cuenta previa: se crea al iniciar sesión, con 20 búsquedas gratis y sin
tarjeta.

## App de Claude (claude.ai, escritorio y móvil)

1. Configuración → **Plugins** → Agregar → **Agregar desde un repositorio**.
2. Pega `EmilianoMierUjed/skills_Consulto` y confirma.
3. Instala **Consulto** de la lista.
4. La primera vez que lo uses, Claude abre el inicio de sesión: entra con Google.

Si te ofrece **sincronización automática con GitHub**, puedes saltarla: solo sirve para
recibir actualizaciones solas y requiere darle acceso a la app de GitHub.

Funciona también en el plan gratuito de Claude. Conectado una vez desde la web, el
conector sirve igual en la app móvil.

## Claude Code

```
/plugin marketplace add EmilianoMierUjed/skills_Consulto
/plugin install consulto-legal-mcp@consulto
```

La primera vez ejecuta `/mcp` para completar el inicio de sesión en el navegador.

Solo el conector, sin skills:

```
claude mcp add --transport http consulto https://mcp.consulto.page/mcp
```

## Otros clientes MCP (Cursor, Codex, opencode, ChatGPT…)

Pega el endpoint `https://mcp.consulto.page/mcp` en la configuración MCP de tu
herramienta; el inicio de sesión OAuth se abre solo. Las skills de este repositorio
(`skills/*/SKILL.md`) sirven como instrucciones de metodología en cualquier asistente.

Guía paso a paso por herramienta: https://consulto.page/conectar

## Las 4 skills

| Skill | Qué hace |
|---|---|
| `jurisprudencia` | Búsqueda SCJN con priorización de criterios obligatorios por circuito y links al Semanario |
| `sentencias` | Búsqueda de engroses SCJN con snippets acotados, links oficiales y vínculo tesis→sentencia |
| `corpus-iuris` | Estándares internacionales separados por valor vinculante, con URL oficial en cada fuente |
| `control-difuso` | Metodología de 4 pasos de constitucionalidad y convencionalidad con citas verificables |

## Licencia

Apache-2.0.
