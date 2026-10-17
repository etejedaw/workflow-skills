# Changelog

Formato basado en [Keep a Changelog](https://keepachangelog.com/es-ES/1.1.0/).

## [Unreleased]

### Added

- `git-commit`, migrada desde la skill `commit` de completr-node-monorepo.
- `release-changelog`, migrada desde la skill `changelog` de completr-node-monorepo, con rutas configurables y sin el changelog público propio de completr.
- `sequelize-migration`, generalizada desde la skill `db-migration` de completr-node-monorepo.
- `typeorm-migration`, basada en el flujo de migraciones de contabilidad-tejeda-backend, con la regla de `timestamptz` para las fechas en PostgreSQL.
- `bruno-endpoints`, generalizada desde la skill `endpoint-doc` de completr-node-monorepo, con soporte para Express, Hono y NestJS.
