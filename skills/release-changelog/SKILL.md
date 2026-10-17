---
name: release-changelog
description: Generate per-feature technical changelog documents (default docs/changelogs/) summarizing commits since the last documented release, plus public release entries when the project defines them and the changes are user-facing. Trigger when the user asks to "registrar cambios", "actualizar el changelog", "preparar release", "documentar lo que se ha hecho" o antes de un paso a productivo. Never rewrites historical entries.
license: Apache-2.0. Complete terms in LICENSE
---

# Release Changelog

Genera dos tipos de changelog:

- **Técnico**: un archivo por feature (Keep a Changelog adaptado), para quien lee el código. Formato en [`changelog-template.md`](./changelog-template.md).
- **Público** (opcional): una entrada por release visible para los usuarios del producto. Solo existe si el proyecto lo define (ver "Configuración del proyecto").

La skill define el **proceso**: qué commits agrupar, qué descartar, cómo nombrar y cuándo hay entrada pública. Lee la plantilla antes de generar.

`Released`, `Summary` y `Commits` son obligatorias en cada archivo técnico; el resto solo si aplica.

## Configuración del proyecto

Antes de empezar, lee el `CLAUDE.md` del proyecto y los documentos que enlace. Lo que defina el proyecto manda sobre estos valores por defecto:

| Ajuste | Por defecto |
| --- | --- |
| Carpeta del changelog técnico | `docs/changelogs/` |
| Rama base para comparar | `main` |
| Idioma del changelog técnico | El mismo que el resto de `docs/` |
| Changelog público | No existe |

Si el proyecto tiene changelog público, su `CLAUDE.md` debe indicar:

- La carpeta de las entradas (ej: `apps/landing/src/content/releases/`).
- Dónde vive el schema o formato de cada entrada, y el comando que lo valida (ej: el build de la landing).
- El idioma de cada campo y los nombres de secciones que ve el usuario, si difieren de los internos.

Si el proyecto no dice nada del changelog público, no lo generes ni lo propongas.

## Contexto general

- **Un solo changelog técnico para todo el repo.** En un monorepo, un grupo puede mezclar commits de distintas apps si cuentan la misma feature.
- **El changelog público es un subconjunto.** Todo grupo tiene su archivo técnico; solo los grupos con cambios que un usuario nota tienen además entrada pública.
- **Si el repo es público, los changelogs también.** No mencionan herramientas ni documentos internos de planificación, solo lo que cambió en el código y en `docs/`.
- **Estilo del técnico:** sin emojis, frases cortas, mismo tono que el resto de `docs/`.

## Cuándo invocar

- **Antes de un paso a productivo**: documentar todo lo que entra al deploy.
- Tras completar una feature significativa: nuevo módulo, refactor mayor, cambio de convención, nueva integración con un servicio externo, cambio en el sistema de errores.
- Cuando el usuario lo pide explícitamente ("actualiza el changelog", "documenta lo que se hizo").

## Información que necesitas antes de empezar

1. **Rama base** para comparar. Si la rama actual es la base, preguntar la referencia (último tag, hash específico, último merge de release).
2. **Rama actual**: la que tiene los commits a documentar (típicamente `develop` o una feature branch).

Si todos los commits desde la base ya están documentados en algún changelog, la skill avisa y termina sin generar nada.

## Flujo

### 1. Cargar contexto

- Lista los changelogs existentes en la carpeta del técnico y extrae los hashes documentados con grep por la sección `## Commits`.
- Si la carpeta no existe, es el primer changelog del proyecto: créala.

### 2. Listar commits no documentados

```bash
git log <base>..HEAD --pretty=format:"%h %ad %s" --date=short
```

Compara con los hashes ya documentados. Los que sobran son candidatos a entrar en algún changelog.

### 3. Filtrar ruido

**NUNCA** incluir:

- Bookkeeping de tareas o notas (`docs(todo): mark X as completed`, cambios que solo tocan `TODO.md`, `MEMORY.md` o similares).
- Style/format puro **standalone** (si acompaña a un feature, va con el feature).
- Merge commits.
- Commits que solo actualizan la propia carpeta de changelogs.

Si un commit es ambiguo, preguntar al usuario antes de descartarlo.

### 4. Agrupar por feature

Analiza subject + archivos tocados de cada commit. Agrupa por:

- **Módulo de negocio nuevo o tocado**, junto con la parte del frontend que lo consume si está en el mismo repo.
- **Feature del frontend sin cambios en el backend.**
- **Integración nueva o ajustada** con un servicio externo.
- **Cambio transversal** (sistema de errores, convención de imports, middleware nuevo).
- **Documentación arquitectónica** (cambios estructurales en `docs/` que valga la pena destacar).

**Granularidad:** una feature = algo que merece un doc autocontenido. Si varios commits cuentan una sola historia ("agregamos el endpoint X, ajustamos el schema, documentamos en Bruno"), un solo changelog. Si son cambios pequeños independientes, agruparlos en un doc `refinements`.

Casos típicos:

- 5 commits implementando un módulo (entidad + migración + service + controller + docs de la API) → 1 doc.
- 1 commit aislado renombrando un schema → probablemente parte de un `refinements`, no su propio archivo.
- Cambio en una convención documentada + ajustes en 3 módulos para cumplirla → 1 doc con sección `Migration` explicando el cambio.

### 5. Proponer y confirmar grupos

Mostrar al usuario:

- Cantidad de grupos identificados.
- Nombre tentativo de cada uno (`YYYY-MM-DD-<feature>.md`).
- Cuántos commits incluye cada uno y la fecha calculada.
- Commits descartados (con razón breve).
- Si el proyecto tiene changelog público: qué grupos llevan **entrada pública** y cuáles no, con la razón. Si varios grupos van a una sola entrada (por ejemplo, fixes sueltos), dilo.

**Esperar confirmación.** Si el usuario reagrupa, ajustar y volver a mostrar.

### 6. Generar los archivos técnicos

Por cada grupo confirmado, crear `<carpeta>/YYYY-MM-DD-<feature>.md` siguiendo [`changelog-template.md`](./changelog-template.md). Secciones obligatorias: `Released`, `Summary`, `Commits`. Las secciones `Added / Changed / Removed / Fixed / Migration` solo si aplican.

**Estilo:**

- Tono profesional, sin emojis, bullets concretos y frases cortas.
- Hashes con backticks: `` `c892226` `` — subject del commit.
- Referenciar módulos y features por su nombre real en el código (carpeta o módulo), no por apodos.
- En la sección `Commits`, listar **todos** los commits del grupo (incluso style/refactor accesorios): ese es el ledger.
- En `Released`: fecha `YYYY-MM-DD` del último commit del grupo.

### 7. Generar la entrada pública (solo si el proyecto la define)

**Cuándo lleva entrada pública:** cuando el usuario del producto nota el cambio. Por ejemplo, una feature nueva, un cambio de UX, una mejora visible de rendimiento o un bug que el usuario sufría.

**Cuándo no:** refactors, tooling, dependencias, CI, Docker, deploy, documentación y cambios internos sin efecto visible. Ante la duda, pregunta en el paso 5.

**Agrupación:** normalmente un grupo técnico da una entrada pública. Varios grupos pequeños que el usuario percibe como "arreglos" van juntos en una sola entrada. Una versión grande puede tener además su propia entrada de resumen.

**Formato:** el que defina el proyecto. Sigue su schema al pie de la letra y toma como referencia de estructura y estilo las entradas más recientes de la carpeta. La fecha es la misma que el `Released` del changelog técnico.

**Tono:** para usuarios, no para devs. Sin nombres de endpoints, módulos, tablas, commits ni librerías. En segunda persona o impersonal ("You can now…", "Ahora puedes…"). Usa los nombres que ve el usuario en la app, no los internos.

**Mostrar el texto al usuario antes de dejarlo listo.** El changelog público es copy de producto y se revisa con el usuario igual que los grupos del paso 5.

### 8. Verificar

- `git log <base>..HEAD --oneline | wc -l` coincide con (commits documentados + commits descartados).
- Ningún changelog viejo fue tocado, ni técnico ni público.
- Si hubo entrada pública: el comando de validación del proyecto pasa.

**No hace commit automáticamente.** Deja los archivos en el working tree para que el usuario revise y use la skill `git-commit`.

## Decisiones que toma esta skill (sin preguntar)

- Excluir todo bookkeeping y los commits de solo estilo standalone.
- Fecha del archivo = fecha del último commit del grupo (formato `YYYY-MM-DD`).
- Nombre del archivo: kebab-case derivado del tema dominante del grupo.
- Estructura de `changelog-template.md` (sin inventar secciones nuevas).
- Que un grupo sin cambios visibles para el usuario no lleva entrada pública.

## Decisiones que NUNCA toma esta skill (siempre pregunta)

- Si dos grupos parecen separables o consolidables.
- Si un commit puede ir en dos grupos (¿cuál es el principal?).
- Si un cambio pequeño merece su propio archivo o entra en un `refinements`.
- Si descartar un commit que parece bookkeeping pero podría no serlo.
- Si un cambio dudoso merece entrada pública, y si va sola o agrupada.
- El texto final de la entrada pública.

## Reglas

- **Nunca tocar changelogs ya creados** (técnicos ni públicos). Si una convención cambia, crear un **nuevo** changelog. Los viejos son historia y deben quedar fieles a la fecha que llevan.
- **El changelog se commitea con el feature, no después.** Idealmente: commits del feature + changelog + commit `docs(changelogs): add <feature> entry`. La skill genera los archivos; el usuario los commitea.
- **Los hashes deben coincidir con commits reales.** Si un commit cambia de hash (rebase, amend), actualizar el changelog antes de mergear.

## Anti-patterns

- Un changelog por commit (ruido).
- Un changelog genérico "Misc updates" que mezcla features no relacionadas.
- Reescribir un changelog histórico para reflejar cambios posteriores.
- Incluir commits de bookkeeping.
- Inventar fechas que no correspondan al último commit del grupo.
- Usar emojis en los archivos generados.
- Entrada pública para cambios internos (refactor, tooling, deploy) o con jerga técnica.

## Checklist final

- [ ] Cada commit no documentado entra en algún changelog o está explícitamente descartado.
- [ ] Los hashes de cada doc coinciden con `git log --oneline`.
- [ ] No se modificó ningún changelog viejo.
- [ ] Si el proyecto tiene changelog público: cada grupo con cambios visibles tiene su entrada, o el usuario decidió que no la lleve, y la validación del proyecto pasa.
- [ ] El usuario revisó los archivos antes de commitear.
