---
type: future-feature
title: "Futuro: Expansiones de Juegos de Mesa (Board Game Expansions)"
description: "Diseño de expansiones enlazadas a su juego base"
tags: [futuro]
---

# Futuro: Expansiones de Juegos de Mesa (Board Game Expansions)

**Fase:** sin asignar
**Estado:** ✅ **Implementado.** Diseño cerrado con el usuario el 2026-10-18; backend en el PR #496 (2026-10-18) y frontend en el PR #563 (2026-10-19). Este documento se conserva como registro del diseño acordado, no como propuesta pendiente.

## Descripción

Hoy un juego de mesa es una entrada plana: no hay forma de registrar que tienes *Catan* y además
*Catan: Navegantes*. Las expansiones existen en BoardGameGeek como entidades propias (tipo
`boardgameexpansion`), pero el backend no las ve en absoluto.

El objetivo es que, desde la ficha de un juego base, un botón **«Añadir expansiones»** liste las
expansiones que BGG conoce de ese juego y permita añadir las seleccionadas. Cada expansión es una
**entrada propia** en la colección: tiene su estado, su nota, su valoración personal y sus partidas,
igual que cualquier otro juego.

### Decisiones cerradas con el usuario

| Pregunta | Decisión |
|---|---|
| ¿Qué es una expansión en la colección? | **Entrada propia** enlazada a su juego base (no un simple checklist informativo) |
| ¿Dónde se ve? | En la lista general **y** en un bloque dentro de la ficha del base; con badge y filtro |
| ¿Se puede añadir sin el base? | **No.** El base debe estar en la colección |
| ¿Qué cuenta como «tener el base»? | **Solo `OWNED`.** `WISHLIST` no basta |
| Estado de las expansiones al añadirlas | `OWNED`, sin precio de adquisición (alta masiva) |
| ¿Qué pasa si BGG falla? | `502` explícito (no lista vacía) |

## Lo que ofrece BGG (verificado el 2026-10-18)

Los *thing types* oficiales del XML API 2 son `boardgame`, `boardgameexpansion`, `boardgameaccessory`, `videogame`, `rpgitem` y `rpgissue`. El tipo viaja en el atributo `type` del elemento `<item>`, tanto en `/thing` como en `/search`.

Para saber **qué expansiones tiene un juego base** se usa el juego base en `/thing` y se leen sus `link` de tipo `boardgameexpansion` **que no lleven `inbound`**:

| Elemento | Significado |
|---|---|
| `<item type="boardgameexpansion">` | Es una expansión |
| `link[@type='boardgameexpansion']` sin `inbound` | Expansión **de** este juego base |
| `link[@type='boardgameexpansion' and @inbound='true']` | Juego base **del que** esta expansión depende |

**Cuidado con la dirección:** leer los links sin filtrar `inbound` invierte la relación. En la ficha de una expansión, sus links `boardgameexpansion` apuntan *hacia su base*, no hacia sub-expansiones.

Ejemplo real del XML de BGG:

```xml
<!-- Juego base: sus expansiones, sin inbound -->
<item type="boardgame" id="205637">
  <link type="boardgameexpansion" id="161936" value="Dunwich Legacy" />
  <link type="boardgameexpansion" id="161937" value="Carcosa" />
</item>

<!-- Expansión: type propio + link inbound hacia su base -->
<item type="boardgameexpansion" id="161936">
  <link type="boardgameexpansion" id="205637" value="Arkham Horror: The Card Game" inbound="true" />
</item>
```

Otros datos útiles: `/search` acepta `type=boardgameexpansion`; `/thing` admite un máximo de **20 ids** por llamada; el endpoint `/collection` **etiqueta mal** las expansiones como `subtype=boardgame` (workaround oficial: `excludesubtype=boardgameexpansion` + una segunda llamada con ese subtype) — pero este proyecto no usa `/collection`.

Detalle ampliado en `research/external-apis/boardgamegeek.md`.

## Estado actual del código (por qué hoy es imposible)

`backend-collection`, `main`:

| Punto | Estado real |
|---|---|
| `BggXmlClient.search()` | Fuerza `queryParam("type", "boardgame")` — **hardcodeado** |
| `BoardGameXmlMapper.BggXmlItem` | **No tiene campo `type`**; el atributo `item@type` se descarta |
| `BoardGameXmlMapper.enrich()` | Extrae `boardgamepublisher`, `boardgamedesigner`, `boardgamecategory`, `boardgamemechanic`… **nunca** `boardgameexpansion` |
| `BggLink` (mapper) | Solo mapea `type` y `value`. **No mapea `id`** ni `inbound` |
| `BoardGameSearchResult` | Sin campo de tipo ni de juego base |
| `BoardGame` / `BoardGameEntity` | Sin campo de expansión |
| `SpringDataBoardGameRepository.findByBggId()` | Devuelve `Optional` y **no se usa en ningún sitio**: en un modelo multi-usuario es incorrecto (dos usuarios pueden tener el mismo `bggId`). No se reutiliza |

El dato, por tanto, **viaja en respuestas que ya pedimos y se descarta**.

## Diseño de datos

Dos campos nuevos en `BoardGame` y `BoardGameEntity` (colección `board_games`) más uno desnormalizado:

| Campo | Tipo | Notas |
|---|---|---|
| `bggType` | enum `BoardGameType { BOARDGAME, BOARDGAME_EXPANSION }` | `null` en los registros existentes **= base**. No requiere migración |
| `parentId` | `String` (id local del registro del base) | `null` en bases. **Inmutable** después de crearse |
| `parentTitle` | `String` | Desnormalizado, para pintar «expansión de *Catan*» en el listado sin N+1. Mismo criterio que `userOwned` |

Índice nuevo: `{ownerId: 1, parentId: 1}` (consulta «mis expansiones de este base»).

Se descarta crear una colección `expansions` aparte: el resto de colecciones ya son registros planos por propietario y una colección nueva duplicaría estado, notas, valoración, `userOwned` y visibilidad sin ganar nada.

Se descarta usar un booleano `expansion` en lugar del enum: BGG tiene más tipos (`boardgameaccessory`, `boardgameintegration`…) y el enum mapea 1:1 sin coste extra.

## Reglas de negocio

| # | Regla | Cuándo se comprueba |
|---|---|---|
| R1 | `BOARDGAME_EXPANSION` implica `parentId != null` | al crear |
| R2 | `BOARDGAME` implica `parentId == null` | al crear |
| R3 | El base debe ser un registro **del propio usuario** con `status = OWNED` | al crear |
| R4 | El `bggId` del base debe estar entre los bases de BGG de esa expansión | al crear |
| R5 | `parentId` no se puede modificar con `PUT /boardgames/{id}` | siempre |
| R6 | Borrar el base borra en cascada sus expansiones | al borrar |
| R7 | Cambiar el base a `WISHLIST` **no** borra nada (R3 se valida al añadir, no se revalida) | — |

R5 es la que compra la simplicidad: como el padre es inmutable y el hijo siempre es un registro nuevo, un ciclo es imposible sin necesidad de detectarlos.

## API

### `GET /boardgames/{id}/expansions` (público, como el resto de GET)

Catálogo de expansiones de BGG para ese base, con el flag de «ya lo tienes» calculado para quien mira.

```json
{
  "baseGameId": "665f1a...",
  "baseBggId": "13",
  "expansions": [
    {
      "bggId": "926",
      "title": "Catan: Navegantes",
      "yearPublished": 1997,
      "imageUrl": "https://...",
      "thumbnailUrl": "https://...",
      "inCollection": true
    }
  ]
}
```

- `inCollection` se calcula **fuera de la caché**, con una sola consulta `findByOwnerIdAndBggIdIn`. Sin sesión, todo `false`.
- `404` si el registro no existe · `502` si BGG falla.

### `POST /boardgames/{id}/expansions` (requiere auth)

```json
{ "bggIds": ["926", "325"] }
```

Respuesta `201`:

```json
{
  "created": [ "…BoardGameResponse" ],
  "skipped": [ { "bggId": "325", "reason": "ALREADY_IN_COLLECTION" } ]
}
```

Es idempotente: lo que ya está en la colección se salta y se informa, sin romper el alta. Lote máximo **20** (límite de `/thing` en BGG).

Validaciones, en orden: el base existe (`404`) → el base es tuyo (`403`) → el base está en `OWNED` (`422`) → cada `bggId` es `boardgameexpansion` en BGG **y** su link `inbound` apunta al `bggId` del base (`422`).

Los registros creados copian el mismo bloque de campos que ya rellena la búsqueda (descripción, año, jugadores, duración, editorial, diseñadores, categorías, mecánicas, imagen, rating), de modo que una expansión se ve igual que cualquier otro juego. Se crean con `status = OWNED`, `dateAdded = hoy` y sin `acquisitionPrice`.

### Modificaciones

| Cambio | Detalle |
|---|---|
| `GET /boardgames` | Nuevo filtro `?type=base\|expansion\|all` (por defecto `all`) |
| `BoardGameResponse` | Añade `bggType`, `parentId` y `parentTitle` |
| `DELETE /boardgames/{id}` | Si es un base, borra en cascada sus expansiones. Sigue devolviendo `204` |
| `POST` y `PUT /boardgames` | **No** aceptan `bggType` ni `parentId`: las expansiones solo nacen del endpoint nuevo, lo que mantiene las invariantes cerradas |

### Trampa de implementación

`PUT /boardgames/{id}` construye el dominio con `mapper.toDomain(request)`. Como `BoardGameRequest` no lleva `parentId` ni `bggType`, **el update los borraría** si no se copian explícitamente del registro existente. Hay que preservarlos.

## Errores

| Código | Cuándo |
|---|---|
| `400` | `bggIds` vacío o con más de 20 elementos |
| `403` | El juego base no pertenece al usuario |
| `404` | El juego base no existe |
| `422` | El base no está en `OWNED`; el `bggId` no es una expansión en BGG; o la expansión no pertenece a ese base |
| `502` | BGG no responde o devuelve error |

El `502` es una **inconsistencia deliberada** con `GET /boardgames/search`, que hoy se traga el error y devuelve `200` con lista vacía. Para un botón eso es inaceptable: hay que distinguir «no hay expansiones» de «BGG no contesta».

## Caché

- Nueva caché Caffeine `boardgameExpansions`, con clave **solo el `bggId` del base** (catálogo puro de BGG, independiente del usuario). El flag `inCollection` se calcula fuera.
- Al crear o borrar expansiones hay que invalidar `boardgameList`, `boardgameDetail` y `stats` siguiendo el patrón de invalidación que ya usa la aplicación.
- La caché `boardgameExpansions` **está implementada** en `CacheConfig.CACHE_NAMES`. El total de cachés es **43** (recuento sobre `origin/main` el 2026-10-19; el checkout local está en una rama obsoleta y da 33).

## Frontend

| Fichero | Cambio |
|---|---|
| `src/types/BoardGame.ts` | Añadir `bggType`, `parentId`, `parentTitle`; nuevo tipo `BoardGameExpansion` |
| `src/api/boardgamesApi.ts` | `getExpansions(id)` y `addExpansions(id, bggIds)` |
| `src/components/AddExpansionsDialog.tsx` (nuevo) | Clonando el patrón de `AcquireBoardGameDialog` (`createPortal`, props `onSave`/`onClose`/`busy`): checkboxes con las expansiones de BGG, las que ya tienes marcadas y deshabilitadas, botón «Añadir N seleccionadas» |
| `src/pages/BoardGameDetailPage.tsx` | Bloque **«Expansiones»** con las que tienes como cards enlazando a su ficha, más el botón «Añadir expansiones» (solo si es base, es tuyo y está en `OWNED`). Si el juego es una expansión, en su lugar muestra «Expansión de *{parentTitle}*» con enlace al base |
| `src/pages/BoardGameListPage.tsx` | Filtro **Todos / Bases / Expansiones** (llama a `?type=`) y badge «Expansión» en las cards |
| `src/components/BoardGameCard.tsx` | Badge, reutilizando el estilo de `BoardGameStatusBadge` |

## Tests

- **Backend** (`mvn verify`, con el umbral JaCoCo del 80 % que ya es obligatorio): parseo de `item@type` y de links con y sin `inbound` en `BoardGameXmlMapperTest`; listado de expansiones en `BggXmlClientTest`; invariantes R1–R3 y cascada R6 en `BoardGameServiceTest`; y los códigos `403`/`404`/`422`/`502` en `BoardGameControllerTest`.
- **Frontend** (`npm test`): nuevo `src/__tests__/AddExpansionsDialog.test.tsx` y actualización de los tests de card, listado y detalle. No hay script de cobertura y no hace falta.

## Fuera de alcance

- ~~Documentar los cambios de esquema en `data-model/schema.md` o en `specs/board-games/`.~~ **Pendiente y ya desbloqueado**: la feature está implementada en `origin/main`, así que `specs/board-games/spec.md`, `specs/board-games/api-contract.md`, `data-model/schema.md` y `architecture/backend.md` deberían reflejarla. Hoy **no** la mencionan.
- Importar la colección de BGG (`/collection`).
- Otros tipos de BGG (`boardgameaccessory`, `boardgameintegration`…): el enum los soporta, pero la UI solo trata bases y expansiones.
- Registrar varias expansiones anidadas: la regla R4 lo permite de forma natural (el padre puede ser cualquier registro tuyo en `OWNED`), pero la UI no lo destaca.

## Issues

| Repo | Issue | Estado |
|---|---|---|
| `backend-collection` | [#495](https://github.com/JavierMedarde99/backend-collection/issues/495) — Expansiones de juegos de mesa (backend): modelo, parser de BGG y endpoints | ✅ Cerrado (PR #496, 2026-10-18) |
| `frontend-collection` | [#562](https://github.com/JavierMedarde99/frontend-collection/issues/562) — Expansiones de juegos de mesa (frontend): bloque en la ficha, diálogo de alta y filtro | ✅ Cerrado (PR #563, 2026-10-19) |

## Prioridad: Media

*Cumplida.* Completa la colección de juegos de mesa, que era la única junto a videojuegos que no podía
representar una relación real entre sus elementos.

---

*Ver también: `specs/board-games/spec.md`, `research/external-apis/boardgamegeek.md`.*
