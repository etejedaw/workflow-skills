---
name: release-changelog
description: Generate per-feature technical changelog documents (default docs/changelogs/) summarizing commits since the last documented release. Trigger when the user asks to "registrar cambios", "actualizar el changelog", "preparar release", "documentar lo que se ha hecho" o antes de un paso a productivo. Never rewrites historical entries.
license: Apache-2.0. Complete terms in LICENSE
---

# Release Changelog

Genera un changelog técnico: un archivo por feature (Keep a Changelog adaptado), para quien lee el código. Formato en [`changelog-template.md`](./changelog-template.md).

La skill define el **proceso**: qué commits agrupar, qué descartar y cómo nombrar. Lee la plantilla antes de generar.

`Released`, `Summary` y `Commits` son obligatorias en cada archivo; el resto solo si aplica.

## Configuración del proyecto

Antes de empezar, lee el `CLAUDE.md` del proyecto y los documentos que enlace. Lo que defina el proyecto manda sobre estos valores por defecto:

| Ajuste | Por defecto |
| --- | --- |
| Carpeta del changelog | `docs/changelogs/` |
| Rama base para comparar | `main` |
| Idioma del changelog | El mismo que el resto de `docs/` |

## Contexto general

- **Un solo changelog para todo el repo.** En un monorepo, un grupo puede mezclar commits de distintas apps si cuentan la misma feature.
- **Si el repo es público, los changelogs también.** No mencionan herramientas ni documentos internos de planificación, solo lo que cambió en el código y en `docs/`.
- **Estilo:** sin emojis, frases cortas, mismo tono que el resto de `docs/`.

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

- Lista los changelogs existentes en la carpeta y extrae los hashes documentados con grep por la sección `## Commits`.
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

**Esperar confirmación.** Si el usuario reagrupa, ajustar y volver a mostrar.

### 6. Generar los archivos

Por cada grupo confirmado, crear `<carpeta>/YYYY-MM-DD-<feature>.md` siguiendo [`changelog-template.md`](./changelog-template.md). Secciones obligatorias: `Released`, `Summary`, `Commits`. Las secciones `Added / Changed / Removed / Fixed / Migration` solo si aplican.

**Estilo:**

- Tono profesional, sin emojis, bullets concretos y frases cortas.
- Hashes con backticks: `` `c892226` `` — subject del commit.
- Referenciar módulos y features por su nombre real en el código (carpeta o módulo), no por apodos.
- En la sección `Commits`, listar **todos** los commits del grupo (incluso style/refactor accesorios): ese es el ledger.
- En `Released`: fecha `YYYY-MM-DD` del último commit del grupo.

### 7. Verificar

- `git log <base>..HEAD --oneline | wc -l` coincide con (commits documentados + commits descartados).
- Cada hash de la sección `Commits` existe en el repo: `git cat-file -e <hash>^{commit}` no debe fallar. Si alguno falla, el commit cambió de hash (rebase, amend) y hay que corregirlo antes de entregar.
- Ningún changelog viejo fue tocado.

**No hace commit.** Deja los archivos en el working tree para que el usuario los revise.

## Decisiones que toma esta skill (sin preguntar)

- Excluir todo bookkeeping y los commits de solo estilo standalone.
- Fecha del archivo = fecha del último commit del grupo (formato `YYYY-MM-DD`).
- Nombre del archivo: kebab-case derivado del tema dominante del grupo.
- Estructura de `changelog-template.md` (sin inventar secciones nuevas).

## Decisiones que NUNCA toma esta skill (siempre pregunta)

- Si dos grupos parecen separables o consolidables.
- Si un commit puede ir en dos grupos (¿cuál es el principal?).
- Si un cambio pequeño merece su propio archivo o entra en un `refinements`.
- Si descartar un commit que parece bookkeeping pero podría no serlo.

## Reglas

- **Nunca tocar changelogs ya creados.** Si una convención cambia, crear un **nuevo** changelog. Los viejos son historia y deben quedar fieles a la fecha que llevan.
- **Los hashes deben coincidir con commits reales.** Si un commit cambia de hash (rebase, amend), actualizar el changelog antes de mergear.

## Anti-patterns

- Un changelog por commit (ruido).
- Un changelog genérico "Misc updates" que mezcla features no relacionadas.
- Reescribir un changelog histórico para reflejar cambios posteriores.
- Incluir commits de bookkeeping.
- Inventar fechas que no correspondan al último commit del grupo.
- Usar emojis en los archivos generados.

## Checklist final

- [ ] Cada commit no documentado entra en algún changelog o está explícitamente descartado.
- [ ] Los hashes de cada doc existen (`git cat-file -e`) y coinciden con `git log --oneline`.
- [ ] No se modificó ningún changelog viejo.
- [ ] El usuario revisó los archivos.
