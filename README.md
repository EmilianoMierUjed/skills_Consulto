# Consulto — plugin y skills jurídicas

**Consulto** conecta tu asistente de IA (Claude, Claude Code, Codex, opencode, ChatGPT) con
la jurisprudencia de la SCJN (311,537 tesis) y el corpus internacional de derechos humanos
(268,886 documentos: Corte IDH, CIDH, comités ONU, OIT, tratados). Cada respuesta trae su
cita con registro digital y link oficial.

Necesitas una cuenta activa de Consulto.
Guía completa por herramienta: la página **Conectar** del sitio de Consulto.

## Claude Code (recomendado): plugin oficial

Instala el conector MCP y las tres skills en un paso:

```
/plugin marketplace add EmilianoMierUjed/skills_Consulto
/plugin install consulto-legal-mcp@consulto
```

La primera vez ejecuta `/mcp` para completar el inicio de sesión en el navegador
(usa el mismo correo de tu cuenta de Consulto).

Solo el conector, sin skills:

```
claude mcp add --transport http consulto https://137-184-20-161.sslip.io/mcp
```

## App de Claude (claude.ai)

1. Configuración → Conectores → **Agregar conector personalizado**.
2. Nombre: `Consulto` · URL: `https://137-184-20-161.sslip.io/mcp`.
3. Inicia sesión con el correo de tu cuenta de Consulto.
4. Skills: descarga los `.zip` desde el sitio de Consulto y súbelos en
   Configuración → Capacidades → Skills.

Conectado desde la web, el conector también funciona en la app móvil de Claude.

## Otros clientes MCP (Codex, opencode, ChatGPT…)

Pega el endpoint `https://137-184-20-161.sslip.io/mcp` en la configuración MCP de tu
herramienta; el inicio de sesión OAuth se abre solo. Las skills de este repositorio
(`skills/*/SKILL.md`) sirven como instrucciones de metodología en cualquier asistente.

## Las 3 skills

| Skill | Qué hace |
|---|---|
| `jurisprudencia` | Búsqueda SCJN con priorización de criterios obligatorios por circuito y links al Semanario |
| `corpus-iuris` | Estándares internacionales separados por valor vinculante, con URL oficial en cada fuente |
| `control-difuso` | Metodología de 4 pasos de constitucionalidad y convencionalidad con citas verificables |

## Licencia

Apache-2.0.
