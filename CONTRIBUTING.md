# Cómo trabajamos en Canchitas App

Estas reglas aplican a todos los repos de la organización:
`canchitas-api`, `canchitas-mobile`, `canchitas-admin` y `canchitas-landing`.

## Flujo de trabajo

Usamos **trunk-based development**: `main` es la única rama de larga vida y siempre tiene que funcionar. Todo cambio pasa por un Pull Request.

```
issue  →  rama corta desde main  →  commits  →  Pull Request  →  revisión  →  squash merge  →  la rama se borra sola
```

1. **Partí de un issue.** Si no existe, crealo con el formulario que corresponda: bug, feature o tarea.
2. **Actualizá `main` y creá la rama:**
   ```bash
   git switch main && git pull
   git switch -c feat/busqueda-por-barrio
   ```
3. **Commiteá en pasos chicos** con el formato de commits de más abajo.
4. **Subí la rama y abrí el PR:**
   ```bash
   git push -u origin feat/busqueda-por-barrio
   gh pr create --base main
   ```
5. **Mergeá con squash** cuando el PR esté revisado:
   ```bash
   gh pr merge --squash --delete-branch
   git switch main && git pull
   ```

Una rama debería vivir **días, no semanas**. Cuanto más vive, más se aleja de `main` y más difícil es mergearla.

## Nombres de ramas

```
<tipo>/<descripcion-corta-en-kebab-case>
```

| Tipo | Cuándo |
|---|---|
| `feat/` | Funcionalidad nueva |
| `fix/` | Corrección de un bug |
| `refactor/` | Cambio de código que no altera el comportamiento |
| `docs/` | Solo documentación |
| `test/` | Agregar o corregir tests |
| `chore/` | Mantenimiento: dependencias, configuración, tooling |
| `ci/` | Pipelines de CI/CD |
| `perf/` | Mejora de rendimiento |

Ejemplos: `feat/cambiar-password`, `fix/auth-module-circular-import`, `chore/actualizar-nest-11`.

**Ramas de otra persona que quedan guardadas** para retomarlas más adelante llevan el usuario de GitHub de su autor:

```
<tipo>/<usuario>/<descripcion>      ej: feat/augusto-fg/mercadopago
```

Sin espacios, sin mayúsculas, sin tildes ni `ñ` (algunas herramientas no los manejan bien).

## Commits: Conventional Commits

```
<tipo>(<alcance>): <descripción en minúscula, en imperativo, sin punto final>

<cuerpo opcional: POR QUÉ se hizo el cambio, no qué (eso ya lo muestra el diff)>

<pie opcional: Closes #12, BREAKING CHANGE: ...>
```

- **Tipo:** los mismos que las ramas (`feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `ci`, `perf`).
- **Alcance:** la parte del sistema que toca. Por ejemplo, en la API: `auth`, `reservations`, `complexes`, `reviews`, `notifications`, `statistics`, `geocode`; en el mobile: `reservas`, `perfil`, `home`; en cualquier repo: `repo`, `deps`.
- **Primera línea:** 72 caracteres como máximo.

Ejemplos:

```
feat(reservations): permitir cancelar hasta 2 horas antes
fix(auth): revertir import de ReservationsModule en AuthModule
chore(deps): actualizar expo a 54.0.14
docs(repo): documentar variables de entorno en .env.example
```

Si un cambio rompe compatibilidad (por ejemplo, cambia el contrato de un endpoint que usan el mobile o el admin), agregá `!` después del tipo y explicalo en el pie:

```
feat(auth)!: renombrar /auth/profile a /users/me

BREAKING CHANGE: el mobile y el admin tienen que apuntar al endpoint nuevo.
```

> Por qué importa: en la Etapa 2 estos mensajes se validan automáticamente, y más adelante generan el CHANGELOG y la versión de cada release.

## Pull Requests

- **El título del PR sigue el formato de commit**, porque con squash merge **el título del PR pasa a ser el commit que queda en `main`**.
- Completá la plantilla: qué cambia, por qué, cómo se probó.
- Enlazá el issue con `Closes #N`: al mergear, el issue se cierra solo.
- **PRs chicos.** Un PR resuelve una sola cosa. Si toca la API y el mobile, son dos PRs, uno en cada repo, enlazados entre sí.
- Cada repo tiene un `CODEOWNERS` que asigna automáticamente quién revisa.

## `main` protegida

Nadie pushea directo a `main`. Cada repo trae un hook `pre-push` que lo impide localmente. Se activa solo al correr `pnpm install`, porque el script `prepare` configura `core.hooksPath`. Para activarlo a mano:

```bash
git config core.hooksPath .githooks
```

En una emergencia real se puede saltear con `git push --no-verify`, pero queda a la vista en el historial: usalo solo si es imprescindible.

## Secretos y configuración

- **Nunca** se commitea un `.env`. Está en el `.gitignore` de cada repo.
- Cada repo tiene un **`.env.example`** con los nombres de las variables, sin valores. Si agregás una variable nueva, agregala también ahí.
- Si commiteaste un secreto por error, **avisá enseguida**. Borrarlo en un commit nuevo no alcanza: queda en el historial.
- Los falsos positivos de gitleaks revisados a mano se registran en `.gitleaksignore`, con un comentario que explique por qué.

## Saltos de línea

Cada repo tiene un `.gitattributes` que guarda los archivos de texto con **LF** (formato Unix) dentro del repo, aunque trabajes en Windows. Evita diffs que cambian todas las líneas de un archivo sin que cambie nada real, y problemas dentro de Docker.

## Requisitos

- Node.js 24 (LTS)
- pnpm
- Git, con `user.name` y `user.email` configurados:
  ```bash
  git config --global user.name "Tu Nombre"
  git config --global user.email "tu@email.com"
  ```
- En Windows también conviene `git config --global core.longpaths true` (las rutas de `node_modules` de React Native superan el límite de 260 caracteres).
