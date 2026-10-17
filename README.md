# workflow-skills

Skills para mi flujo de desarrollo: commits, changelogs, migraciones de base de datos y documentación de APIs.

Cada skill trae valores por defecto razonables y le da prioridad a lo que diga el `CLAUDE.md` del proyecto (rutas, comandos, idioma), así que la misma skill sirve en repos con estructuras distintas.

## Skills incluidas

### Git

- **`git-commit`** — Commits granulares con Conventional Commits, en una sola línea, en inglés y sin firma de IA. Agrupa los cambios por archivo.
- **`release-changelog`** — Genera changelogs técnicos por feature a partir de los commits no documentados y, si el proyecto lo define, entradas públicas para los usuarios.

### Migraciones

- **`sequelize-migration`** — Crea la migración de Sequelize cuando un cambio en un modelo afecta el esquema. En PostgreSQL usa SQL crudo idempotente.
- **`typeorm-migration`** — Genera la migración de TypeORM desde las entidades y revisa el SQL antes de entregarlo (renombres, enums, ruido de drift).

### Documentación de APIs

- **`bruno-endpoints`** — Mantiene la colección de [Bruno](https://www.usebruno.com) (formato OpenCollection, `.yml`) al día cuando se crea, cambia o elimina un endpoint. Reconoce rutas de Express, Hono y NestJS.

Cada skill es una carpeta con un `SKILL.md` (frontmatter con `name` y `description`, seguido de las instrucciones en Markdown). El formato es compatible con [Claude Code Agent Skills](https://docs.claude.com/en/docs/claude-code/skills).

## Instalación

Con el [CLI de skills.sh](https://skills.sh). Para ver las skills disponibles:

```bash
npx skills add etejedaw/workflow-skills --list
```

Las skills de git sirven en cualquier proyecto, así que conviene instalarlas a nivel de usuario (`-g`):

```bash
npx skills add etejedaw/workflow-skills -g --skill git-commit --skill release-changelog
```

Las de migraciones y Bruno dependen del stack, así que se instalan dentro de cada proyecto, solo las que correspondan:

```bash
npx skills add etejedaw/workflow-skills --skill typeorm-migration --skill bruno-endpoints
```

## Actualización

Las skills instaladas son copias y no se actualizan solas. Para traer los últimos cambios:

```bash
npx skills update -g
npx skills update -p
```

El primer comando actualiza las globales; el segundo, las del proyecto en el que estás.

## Configurar una skill para un proyecto

Si el proyecto no sigue los valores por defecto, decláralo en su `CLAUDE.md`. Por ejemplo:

```markdown
## Skills

- Changelog técnico en `docs/changelogs/`. Changelog público en `apps/landing/src/content/releases/`, con el schema en `apps/landing/src/content.config.ts`; se valida con `npm run build:landing`.
- Migraciones de Sequelize en `apps/api/migrations/`, creadas con `npm run migrate:create -w apps/api -- <nombre>`.
- Los commits van en `develop`.
```

Cada `SKILL.md` lista en su sección "Configuración del proyecto" qué ajustes acepta y cuáles son sus valores por defecto.

## Licencia

Apache 2.0 — ver [LICENSE](LICENSE).
