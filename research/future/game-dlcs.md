---
type: future-feature
title: "Futuro: DLC y Contenido Adicional de Videojuegos (Game DLCs)"
description: "Diseño de DLCs enlazados a su juego base"
tags: [futuro]
---

# Futuro: DLC y Contenido Adicional de Videojuegos (Game DLCs)

**Fase:** sin asignar
**Estado:** 📋 Investigación — **no implementado**. Diseño cerrado con el usuario el 2026-10-19.

## Descripción

Hoy un videojuego es una entrada plana: no hay forma de registrar que tienes *The Witcher 3* y además
sus expansiones *Hearts of Stone* y *Blood and Wine*. RAWG las conoce como entidades propias —tienen su
propio `id`— pero el backend no las ve en absoluto.

El objetivo es que, desde la ficha de un juego base, un botón **«Añadir contenido adicional»** liste lo
que RAWG conoce de ese juego y permita añadir lo seleccionado. Cada DLC es una **entrada propia** en la
colección: tiene su estado, su nota, su valoración y su precio, igual que cualquier otro juego.

### Decisiones cerradas con el usuario

| Pregunta | Decisión |
|---|---|
| ¿Qué es un DLC en la colección? | **Entrada propia** enlazada a su juego base (no un checklist informativo) |
| ¿Dónde se ve? | En la lista general **y** en un bloque dentro de la ficha del base; con etiqueta y filtro |
| ¿Se puede añadir sin el base? | **No.** El base debe estar en la colección |
| ¿Qué cuenta como «tener el base»? | **No estar en `WISHLIST`.** `PLAYING`, `COMPLETED` y `ABANDONED` valen |
| Qué se lista al añadir | **Todo lo que devuelve RAWG**, sin clasificar por heurísticas de nombre |
| Estado de los DLCs al añadirlos | El que se elija en el diálogo, aplicado a todos |
| ¿La búsqueda general los oculta? | **Sí**, vía `exclude_additions` (ver «Modificaciones») |
| Si el DLC ya está suelto en la colección | Se **vincula** al base (no se salta) |
| ¿Qué pasa si RAWG falla? | `502` explícito (no lista vacía) |

## Lo que ofrece RAWG (verificado el 2026-10-19)

Contrastado contra el OpenAPI oficial en `https://api.rawg.io/docs/?format=openapi`. **No se pudo
verificar en vivo** contra una respuesta real: en esta máquina sigue sin haber `RAWG_API_KEY` (la ficha
`rawg.md` ya lo dejaba anotado). Todo lo de abajo está por tanto apoyado en el contrato publicado.

### Sí se puede filtrar: juegos vs adiciones

`GET /games` acepta dos filtros booleanos que hoy no usamos:

| Filtro | Descripción oficial | Qué hace de verdad |
|---|---|---|
| `exclude_additions=true` | «Exclude additions.» | Quita las adiciones → deja juegos base e independientes. **Es el filtro limpio de «solo juegos»** |
| `exclude_parents=true` | «Exclude games which have additions.» | Quita los juegos que *tienen* adiciones → deja las adiciones **y** los juegos independientes. **No es un filtro de «solo DLC»** |

Es decir: «solo juegos» se consigue bien; «solo DLCs» no tiene filtro propio y hay que mirarlos por la vía
del endpoint de adiciones.

### El endpoint de adiciones

`GET /games/{game_pk}/additions` — descripción oficial literal: *«Get a list of DLC's for the game, GOTY
and other editions, companion apps, etc.»*. Parámetros `page` y `page_size` (sin máximo declarado) y
respuesta paginada `{ count, next, previous, results[] }` con elementos del modelo de lista `Game`, así
que cada adición trae su propio `id`, `name` y `background_image`.

El inverso existe también: `GET /games/{game_pk}/parent-games` — *«Get a list of parent games for DLC's
and editions»*, vacío para un juego base. **No se usará** en este diseño.

### Dos avisos importantes

**(a) «additions» no es sinónimo de DLC.** La propia descripción oficial lo dice: mezcla DLC, ediciones
GOTY/Definitiva, **apps compañeras**, etc. No hay ningún campo que los distinga. Decisión cerrada: se
muestra todo tal cual y el usuario elige, sin heurísticas por nombre. Por eso el bloque de la UI se titula
**«Contenido adicional»** y no «DLCs».

**(b) En el detalle sí se puede saber si algo es un DLC; en la lista, no.** `GET /games/{id}` (modelo
`GameSingle`) trae `parents_count` («Parent game count (for DLCs)») y `additions_count` («DLC/addition
count»). El modelo de **lista** no los declara, y el OpenAPI de RAWG es conocido por estar incompleto: ya
está documentado que `genres`, `publishers` y `developers` no aparecen como propiedades y el backend los
lee igualmente — corroborado con un cliente de terceros en producción que también los mapea de la lista.

Para este diseño no es bloqueante: **sabremos que una entrada es un DLC porque la habremos añadido
nosotros** desde el flujo de adiciones. Igual que en las expansiones de BGG.

## Estado actual del código (por qué hoy es imposible)

`backend-collection`, y `frontend-collection`:

| Punto | Estado real |
|---|---|
| `RAWGClient.search()` | Solo envía `search` (+ `key`). **Ni `exclude_*`, ni `page`/`page_size`** |
| `RAWGClient.getGenres()` | Usa `/games/{id}` y lee únicamente `genres[].name` |
| `/games/{pk}/additions` | **No se usa.** La ficha `rawg.md` ya lo listaba como disponible y no usado |
| `Game` / `GameEntity` | Sin ningún campo de relación |
| `GameSearchResult` | Sin tipo ni juego base |
| `GameDtoMapper.toDomain(request)` | El `PUT` reconstruye el dominio desde el DTO: **borraría `parentId`** |
| `SpringDataGameRepository.findByExternalId()` | Devuelve `Optional`, **no se usa en ningún sitio**, y **no está acotado por dueño**: no reutilizarlo |
| `GameService` | No valida `externalId` en absoluto (el `spec` dice que es único, pero el código no lo comprueba) |
| `GameCard.gameToPayload()` (frontend) | El `PUT` manda todos los campos: si `parentId` no viaja, el backend debe preservarlo |

**Precedente a NO imitar:** `BookService.checkExternalIdUnique` comprueba la unicidad de `externalId` de
forma **global, sin filtrar por propietario**, lo que impide que dos usuarios tengan el mismo libro. Aquí
todo se consulta por `ownerId`.

## Diseño de datos

Dos campos nuevos en `Game` y `GameEntity` (colección `games`):

| Campo | Tipo | Notas |
|---|---|---|
| `parentId` | String | Id local del juego base. **Inmutable.** `null` = no es DLC |
| `parentTitle` | String | Denormalizado, para la etiqueta «Contenido adicional de *The Witcher 3*» sin consultas extra. Se pierde exactitud si el base se renombra: se acepta, igual que en las expansiones |

Índice nuevo: `{ownerId: 1, parentId: 1}`.

**Aquí no hace falta un enum de tipo**, a diferencia de las expansiones de BGG: allí el tipo venía en el
XML (`boardgameexpansion`) y había que guardarlo; en RAWG no hay campo fiable que lo diga, así que la
pertenencia la define el hecho de haber entrado por el flujo de adiciones. Un campo menos.

## Reglas de negocio

| # | Regla |
|---|---|
| R1 | Un DLC siempre tiene `parentId`; un juego normal nunca |
| R2 | El base debe ser **del usuario** y **no estar en `WISHLIST`** (`PLAYING`, `COMPLETED` y `ABANDONED` cuentan) |
| R3 | El base debe tener `externalId` con origen RAWG: sin él no hay adiciones que consultar |
| R4 | `parentId` y `parentTitle` **no** se pueden modificar por `PUT /games/{id}` |
| R5 | Borrar el base borra en cascada sus DLCs |
| R6 | Pasar el base a `WISHLIST` después **no** borra nada: R2 se valida al añadir, no se revalida |
| R7 | Idempotencia: lo que ya está en la colección no rompe el alta |

R4 es la que compra la simplicidad: como el padre es inmutable y el hijo siempre nace del mismo flujo, un
ciclo es imposible sin necesidad de detectarlos.

**Consecuencia asumida de vincular (ver API):** al vincular un juego que ya tenías suelto **no se le
cambia el `status`**, así que puede quedar un DLC en `WISHLIST` colgando de un base en `COMPLETED`. Es
correcto —tú ya lo tenías así— pero conviene saberlo.

## API

### `GET /games/{id}/additions` (público, como el resto de GET)

Catálogo de adiciones de RAWG para ese base, con el flag de «ya lo tienes» calculado para quien mira.

```json
{
  "baseGameId": "665f1a...",
  "baseExternalId": "292030",
  "additions": [
    {
      "externalId": "355880",
      "title": "The Witcher 3: Wild Hunt - Blood and Wine",
      "platform": "PC",
      "genres": ["RPG"],
      "thumbnailUrl": "https://media.rawg.io/...",
      "releaseDate": "2016-05-31",
      "externalSource": "RAWG",
      "inCollection": true,
      "alreadyLinked": false
    }
  ],
  "totalAvailable": 12
}
```

- `inCollection`: ¿existe un juego tuyo con ese `externalId`? Se calcula **fuera de la caché**, con una
  sola consulta. Sin sesión, todo `false`.
- `totalAvailable` es el `count` que reporta RAWG; si es mayor que `additions.size()`, la UI avisa de que
  hay más de las que se han traído (ver Caché).
- `404` si el juego no existe · `422` si no es RAWG o no tiene `externalId` · `502` si RAWG falla.

### `POST /games/{id}/additions` (requiere auth)

```json
{ "externalIds": ["355880", "378648"], "status": "PLAYING" }
```

Respuesta `201` con **tres** desenlaces:

```json
{
  "created": [ "…GameResponse" ],
  "linked":  [ "…GameResponse" ],
  "skipped": [ { "externalId": "378465", "reason": "ALREADY_LINKED" } ]
}
```

| Salida | Cuándo |
|---|---|
| `created` | No lo tenías: se crea un DLC nuevo |
| `linked` | Ya lo tenías como juego suelto: se le pone el `parentId` y `parentTitle` y pasa a ser DLC de este base. **Conserva su estado, nota, valoración y precio** |
| `skipped` | Ya está vinculado a este u otro base |

Lote máximo **40** elementos (una página de RAWG). Validaciones en orden: el base existe (`404`) → el base
es tuyo (`403`) → el base no está en `WISHLIST` (`422`) → el base tiene `externalId` de RAWG (`422`) →
cada `externalId` aparece entre las adiciones de ese base en RAWG (`422`).

Los registros de `created` copian el bloque de campos que ya rellena la búsqueda (plataforma, géneros,
miniatura, fecha de estreno, `externalSource = RAWG`, `externalId`), más:

- `status` = el elegido en el diálogo
- `parentId` / `parentTitle` = el base
- `dateAdded` = hoy
- `acquisitionDate` / `acquisitionPrice` = `null` (alta masiva, se relaja la convención de «precio obligatorio al adquirir», como en las expansiones)
- `platform`: si RAWG no devuelve plataforma para la adición, **se hereda la del base** (el campo es obligatorio en el modelo)

### Modificaciones

| Cambio | Detalle |
|---|---|
| `GET /games/search` | Empieza a enviar **`exclude_additions=true`**: la búsqueda deja de ofrecer adiciones. A partir de aquí la vía de entrada de un DLC es la ficha de su base |
| `GET /games` | Nuevo filtro `?type=base\|dlc\|all` (por defecto `all`) |
| `GameResponse` | Añade `parentId` y `parentTitle` |
| `DELETE /games/{id}` | Si es un base, borra en cascada sus DLCs. Sigue devolviendo `204` |
| `POST` y `PUT /games` | **No** aceptan `parentId` ni `parentTitle` |

Efecto secundario asumido: los DLCs añadidos sueltos **antes** de este cambio siguen en la colección; solo
dejan de aparecer en búsquedas nuevas.

### Trampa de implementación

`PUT /games/{id}` construye el dominio con `mapper.toDomain(request)`. Como `GameRequest` no lleva
`parentId` ni `parentTitle`, **el update los borraría** si no se copian explícitamente del registro
existente. Es exactamente la misma trampa que en las expansiones de juegos de mesa. Hay que preservarlos, y
además el frontend reenvía todos los campos en cada `PUT` (`GameCard.gameToPayload`), así que el fallo
aparecería al cambiar cualquier cosa del DLC.

## Errores

| Código | Cuándo |
|---|---|
| `400` | `externalIds` vacío o con más de 40 elementos |
| `403` | El juego base no pertenece al usuario |
| `404` | El juego base no existe |
| `422` | El base está en `WISHLIST`; el base no tiene `externalId` de RAWG; o el `externalId` no es una adición de ese base |
| `502` | RAWG no responde o devuelve error |

El `502` es una **inconsistencia deliberada** con `GET /games/search`, que hoy se traga el error y devuelve
`200` con lista vacía. Para un botón eso es inaceptable: hay que distinguir «no tiene contenido adicional»
de «RAWG no contesta».

## Caché

- Nueva caché Caffeine `gameAdditions`, con clave **solo el `externalId` del base** (catálogo puro de RAWG, independiente del usuario). El flag `inCollection` se calcula fuera.
- Se pide a RAWG con `page_size=40`. Si el `count` que devuelve RAWG supera lo recibido, se refleja en `totalAvailable` para que la UI pueda avisar: no se pagina en esta primera versión (coherente con la limitación de paginación que ya arrastra `RAWGClient`).
- Al crear o borrar DLCs hay que invalidar `gameList`, `gameDetail` y `stats` siguiendo el patrón de invalidación que ya usa la aplicación.
- El total de cachés pasa de **33 a 34** (`CacheConfig.CACHE_NAMES`, verificado el 2026-10-19).

## Frontend

| Fichero | Cambio |
|---|---|
| `src/types/Game.ts` | Añadir `parentId` y `parentTitle`; nuevo tipo `GameAddition` |
| `src/api/gamesApi.ts` | `getGameAdditions(id)` y `addGameAdditions(id, externalIds, status)` |
| `src/components/AddGameDlcsDialog.tsx` (nuevo) | Clonando el patrón de `AcquisitionGameDialog` (`createPortal`, props `onSave`/`onClose`/`busy`): checkboxes con lo que devuelve RAWG, las que ya tienes marcadas y deshabilitadas, **selector de estado** y botón «Añadir N seleccionadas». Distinguir el `502` de «no hay contenido adicional» |
| `src/pages/GameDetailPage.tsx` | Bloque **«Contenido adicional»** con los que tienes como cards enlazando a su ficha, más el botón «Añadir contenido adicional» (solo si es tuyo, no está en `WISHLIST` y tiene `externalId`). Si el juego **es** un DLC, en su lugar muestra «Contenido adicional de *{parentTitle}*» con enlace al base |
| `src/pages/GameListPage.tsx` | Filtro **Todos / Juegos / Contenido adicional** (llama a `?type=`) y etiqueta «DLC» en las cards |
| `src/components/GameCard.tsx` | Etiqueta, reutilizando el estilo de `GameStatusBadge` |

## Tests

- **Backend** (`mvn verify`, con el umbral JaCoCo del 80 % que ya es obligatorio): parseo de `/games/{id}/additions` en `RAWGClientTest`; reglas R1–R3 y la cascada R5 en `GameServiceTest`; los tres desenlaces `created`/`linked`/`skipped` y los códigos `403`/`404`/`422`/`502` en `GameControllerTest`.
- **Frontend** (`npm test`): nuevo `src/__tests__/AddGameDlcsDialog.test.tsx`; actualizar los tests de card, listado y detalle. El repo no tiene script de cobertura y no hace falta.

## Fuera de alcance

- **NO tocar** `specs/games/spec.md`, `specs/games/api-contract.md` ni `data-model/schema.md`: se actualizan cuando el backend implemente de verdad los campos.
- Clasificar adiciones por tipo (DLC / edición / app): no hay dato que lo permita y se decidió no usar heurísticas.
- `GET /games/{pk}/parent-games` (la dirección inversa) y `/game-series`.
- Paginar el listado de adiciones.
- Usar `parents_count`/`additions_count` del detalle de RAWG para etiquetar.
- Sustituir FreeToGame: los juegos de ese origen no tienen adiciones y el endpoint responde `422`, sin romper nada.

## Issues

| Repo | Issue | Estado |
|---|---|---|
| `backend-collection` | [#505](https://github.com/JavierMedarde99/backend-collection/issues/505) — DLC y contenido adicional de videojuegos (backend): modelo, adiciones de RAWG y vinculación | 📋 Abierto |
| `frontend-collection` | [#564](https://github.com/JavierMedarde99/frontend-collection/issues/564) — DLC y contenido adicional de videojuegos (frontend): bloque en la ficha, diálogo de alta y filtro | 📋 Abierto (depende de #505) |

## Prioridad: Media

Completa la colección de videojuegos, la única junto a juegos de mesa que no puede representar una
relación real entre sus elementos. No bloquea nada existente y el trabajo es acotado: dos campos, un
endpoint de lectura, uno de escritura y un diálogo.

---

*Ver también: `specs/games/spec.md`, `research/external-apis/rawg.md`, `research/future/boardgame-expansions.md` (mismo patrón para juegos de mesa).*
