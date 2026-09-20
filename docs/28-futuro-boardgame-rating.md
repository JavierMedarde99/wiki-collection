# Futuro — Calificación personal en juegos de mesa (BoardGames)

## Estado: Investigación / No implementado

## 1. Resumen

Añadir dos campos de evaluación personal a `BoardGame`:

| Campo | Tipo | Validación | Inspiración |
|-------|------|------------|-------------|
| `userRating` | `Integer` | 1-5 (o 0 = sin valorar) | Igual que `Game.userRating` |
| `comment` | `String` | texto libre | Igual que `Game.comment` y `Book.comment` |

**Regla de negocio:** solo tiene sentido valorar juegos con `status = OWNED`. Un juego en `WISHLIST` no se ha jugado todavía, así que no debería tener rating ni comentario.

**Qué NO cambia:** el `bggRating` (puntuación promedio de BGG, 0-10) se mantiene como dato externo de referencia. Es independiente de la valoración personal del usuario.

---

## 2. Estado actual

### 2.1 BoardGame — sin calificación personal

**`BoardGame.java`** (líneas 20-41):
```java
private String id;
private String title;
// ... campos descriptivos ...
private BigDecimal bggRating;   // rating de BGG (externo, 0-10)
private String bggId;
private BoardGameStatus status;  // OWNED | WISHLIST
private String notes;            // notas personales (texto libre, sin valoración numérica)
private LocalDate dateAdded;
```

**`BoardGameEntity.java`**: igual, sin `userRating` ni `comment`.

**`BoardGameRequest.java`**: no tiene `userRating` ni `comment`.

**`BoardGameResponse.java`**: no tiene `userRating` ni `comment`.

**`BoardGameService.java`** (`copyUpdatableFields`): no copia `userRating` ni `comment` porque no existen.

### 2.2 Comparación con Games y Books (que ya tienen)

| Campo | Game.java | Book.java | BoardGame.java (actual) |
|-------|-----------|-----------|------------------------|
| Valoración numérica | `Integer userRating` (1-5) | `Integer start` (0-5) | ❌ No existe |
| Comentario | `String comment` | `String comment` | ❌ No existe (solo `notes`) |
| Validación | `@Min(1) @Max(5)` | `@Min(0) @Max(5)` | N/A |

**Diferencias notables:**
- `Game` usa `userRating` (1-5, mínimo 1). Un juego sin valorar tiene `userRating = null` (campo nullable).
- `Book` usa `start` (0-5, mínimo 0). El 0 significa "sin valorar" explícitamente.
- `BoardGame` propuesta: usar `Integer userRating` con rango 0-5 (`0 = sin valorar`), consistente con `Book.start` y permitiendo explícitamente "no calificado".

### 2.3 BoardGameStatus

**`BoardGameStatus.java`** (enum actual):
```java
public enum BoardGameStatus {
    OWNED,
    WISHLIST
}
```

Solo `OWNED` y `WISHLIST`. No hay estados como `PLAYING` o `COMPLETED` como en Games. El rating solo aplica cuando `status == OWNED`.

---

## 3. Cambios backend

### 3.1 Domain model

**`domain/model/BoardGame.java`** — añadir:
```java
private Integer userRating;   // 0-5, 0 = sin valorar (igual que Book.start)
private String comment;       // notas personales del usuario
```

### 3.2 Entity

**`infrastructure/adapter/out/persistence/BoardGameEntity.java`** — añadir:
```java
private Integer userRating;
private String comment;
```

### 3.3 DTOs

**`infrastructure/adapter/in/web/dto/BoardGameRequest.java`** — añadir:
```java
@Min(value = 0, message = "La puntuación mínima es 0")
@Max(value = 5, message = "La puntuación máxima es 5")
Integer userRating,

String comment,
```

**`infrastructure/adapter/in/web/dto/BoardGameResponse.java`** — añadir:
```java
Integer userRating,
String comment,
```

### 3.4 Mapper

**`infrastructure/adapter/in/web/dto/BoardGameDtoMapper.java`** — añadir mapeo de `userRating` y `comment` en `toDomain()` y `toResponse()`.

**`infrastructure/adapter/out/persistence/BoardGameEntityMapper.java`** — añadir mapeo de `userRating` y `comment` en `toEntity()` y `toDomain()`.

### 3.5 Service

**`application/service/BoardGameService.java`** — `copyUpdatableFields()` añadir:
```java
target.setUserRating(source.getUserRating());
target.setComment(source.getComment());
```

### 3.6 Controller

**`infrastructure/adapter/in/web/BoardGameController.java`**: no requiere cambios — el `PUT /{id}` ya acepta `BoardGameRequest` y el nuevo campo se recibe automáticamente.

### 3.7 MongoDB

El campo se añade a la colección `board_games` como `userRating: Integer` y `comment: String`. No requiere índice nuevo.

### 3.8 Validación: rating solo para OWNED

La validación del rango (0-5) se hace con Bean Validation (`@Min`/`@Max`). La regla de negocio "solo OWNED puede tener rating" es opcional — se puede validar en `BoardGameService.save()`/`update()` o se puede dejar que el frontend maneje esta lógica.

**Opción A (validación backend):**
```java
// En BoardGameService.update() o save()
if (boardGame.getUserRating() != null && boardGame.getUserRating() > 0
    && boardGame.getStatus() != BoardGameStatus.OWNED) {
    throw new IllegalArgumentException("Solo los juegos en OWNED pueden tener valoración personal");
}
```

**Opción B (solo frontend):** el frontend oculta el campo de rating cuando `status !== OWNED`. El backend acepta cualquier valor sin validar el estado.

**Recomendación:** Opción A (validación backend) para consistencia, pero con un mensaje de error claro. Opcionalmente, si el usuario cambia de WISHLIST a OWNED, se le puede permitir añadir rating en el mismo paso (o no).

---

## 4. Frontend

### 4.1 Tipos TypeScript

**`src/types/BoardGame.ts`** — añadir:
```typescript
export interface BoardGame {
  // ... campos existentes ...
  userRating?: number;   // 0-5, undefined = sin valorar
  comment?: string;
}
```

**`src/types/BoardGameFormData.ts`** (o donde esté el tipo de formulario) — añadir:
```typescript
userRating?: number;
comment?: string;
```

### 4.2 Componente: `BoardGameCard.tsx`

Actualizar para mostrar:
- Si `userRating > 0`: mostrar `<StarRating value={boardGame.userRating} readOnly />` (igual que GameCard).
- Si `comment`: mostrar debajo del rating o en sección de detalles.

### 4.3 Componente: `BoardGameForm.tsx`

Añadir campos condicionados:
- **Rating:** solo visible cuando `status === 'OWNED'`. Si el usuario cambia de WISHLIST a OWNED, aparece el campo de rating automáticamente (o el usuario tiene que hacer click en "editar" después).
- **Comentario:** igual, solo visible cuando `status === 'OWNED'` y `userRating > 0` (o siempre visible cuando status=OWNED, como textarea).

**Lógica del formulario:**
```typescript
const showRatingAndComment = form.status === 'OWNED';
```

Si el juego está en WISHLIST y el usuario cambia a OWNED:
- Opción 1: el rating se resetea a 0 automáticamente (pedir al usuario que lo valore después).
- Opción 2: conservar el rating anterior si ya tenía uno (si se editaba antes de cambiar a WISHLIST, lo cual no debería ser posible).

**Recomendación:** resetea a 0 al pasar de WISHLIST a OWNED. Si el usuario vuelve a OWNED después de haber valorado, conserva la valoración anterior (si se guardó antes de cambiar a WISHLIST).

### 4.4 Página: `BoardGameDetailPage.tsx`

Mostrar en sección de detalles:
- `bggRating` (rating externo de BGG, 0-10) — ya visible, mantener.
- `userRating` (rating personal, 0-5) — nuevo, mostrar con `StarRating`.
- `comment` — nuevo, mostrar como texto debajo del rating.

Ejemplo visual:
```
Rating BGG: ⭐ 7.5/10  (externo)
Tu valoración: ⭐⭐⭐⭐☆ 4/5  (personal)
Comentario: "Juego excelente para 4 jugadores, las negociaciones son lo mejor."
```

### 4.5 API client

**`src/api/boardgamesApi.ts`**: no requiere cambios — el `PUT` y `POST` envían `BoardGameFormData` que ya incluye los nuevos campos cuando el frontend los añade al payload.

---

## 5. Migración de datos existentes

No hay migración necesaria — los juegos de mesa existentes simplemente tendrán `userRating = null` y `comment = null` hasta que el usuario los edite y añada valoración. No hay datos previos de rating personal que migrar.

---

## 6. Tests

### Backend

- **`BoardGameDtoMapperTest`**: verificar mapeo de `userRating` y `comment` domain ↔ request/response.
- **`BoardGameEntityMapperTest`**: verificar mapeo entity ↔ domain con los nuevos campos.
- **`BoardGameServiceTest`**: verificar que `copyUpdatableFields` copia los nuevos campos; opcionalmente verificar validación de "solo OWNED puede tener rating".
- **`BoardGameControllerTest`**: verificar que PUT/POST aceptan los nuevos campos sin error 400.

### Frontend

- **`BoardGameCard.test.tsx`**: verificar que el star rating aparece cuando `userRating > 0` y no aparece cuando es 0/null.
- **`BoardGameForm.test.tsx`**: verificar que el campo de rating aparece solo cuando `status === 'OWNED'`; verificar reset a 0 al cambiar de WISHLIST.
- **`BoardGameDetailPage.test.tsx`**: verificar que se muestran `userRating` y `comment` en la sección de detalles.

---

## 7. Esquema de cambios — resumen por archivo

### Backend

| Archivo | Cambio |
|---------|--------|
| `domain/model/BoardGame.java` | Añadir `Integer userRating`, `String comment` |
| `infrastructure/adapter/out/persistence/BoardGameEntity.java` | Añadir `Integer userRating`, `String comment` |
| `infrastructure/adapter/in/web/dto/BoardGameRequest.java` | Añadir `Integer userRating` con `@Min(0) @Max(5)`, `String comment` |
| `infrastructure/adapter/in/web/dto/BoardGameResponse.java` | Añadir `Integer userRating`, `String comment` |
| `infrastructure/adapter/in/web/dto/BoardGameDtoMapper.java` | Mapear `userRating` y `comment` en `toDomain()` y `toResponse()` |
| `infrastructure/adapter/out/persistence/BoardGameEntityMapper.java` | Mapear `userRating` y `comment` en `toEntity()` y `toDomain()` |
| `application/service/BoardGameService.java` | Añadir `setUserRating()` y `setComment()` en `copyUpdatableFields()` |
| `BoardGameController.java` | Sin cambios necesarios |

### Frontend

| Archivo | Cambio |
|---------|--------|
| `src/types/BoardGame.ts` | Añadir `userRating?: number`, `comment?: string` |
| `src/types/BoardGameFormData.ts` | Añadir `userRating?: number`, `comment?: string` |
| `src/components/BoardGameCard.tsx` | Mostrar `StarRating` cuando `userRating > 0` |
| `src/components/BoardGameForm.tsx` | Añadir campos de rating + comentario condicionados a `status === 'OWNED'` |
| `src/pages/BoardGameDetailPage.tsx` | Mostrar `userRating` (star rating) y `comment` en sección de detalles |
| `src/api/boardgamesApi.ts` | Sin cambios necesarios |
| `src/constants/boardGames.ts` | Sin cambios (el enum `BoardGameStatus` ya tiene OWNED/WISHLIST) |

---

## 8. Flujo de usuario

### 8.1 Valorar un juego al añadirlo

```
BoardGameCreatePage
├── Título: "Catan"
├── ...
├── Estado: OWNED  ← al elegir OWNED, aparecen los campos de rating
│   ├── Tu valoración: ⭐⭐⭐☆☆ 3/5
│   └── Comentario: "Juego familiar genial, las negociaciones son lo mejor"
└── [Crear juego de mesa]
```

Si el usuario elige `WISHLIST`, los campos de rating y comentario no aparecen.

### 8.2 Valorar un juego existente

```
BoardGameDetailPage
├── ...
├── Rating BGG: ⭐ 7.5/10 (externo, de BoardGameGeek)
├── Tu valoración: ⭐⭐⭐⭐☆ 4/5 (personal)
├── Comentario: "Juego excelente para 4 jugadores, las negociaciones son lo mejor."
└── [Editar]

→ Naviga a BoardGameEditPage
→ Cambia rating a 5/5, actualiza comentario
→ Guarda
→ Vuelve a BoardGameDetailPage con los nuevos valores
```

### 8.3 Mover de WISHLIST a OWNED

```
BoardGameEditPage (juego en WISHLIST, sin rating)
├── Estado: [WISHLIST → OWNED]
│   → Al cambiar a OWNED, aparece el campo "Tu valoración"
│   → Rating inicial: 0 (sin valorar)
├── ... resto campos ...
└── [Guardar]

→ El juego ahora está en OWNED, el usuario puede valorarlo en la siguiente edición
```

---

## 9. Riesgos y consideraciones

### 9.1 Ratings inconsistency

Un usuario podría tener un juego en OWNED con `userRating = null` (no valorado) y otro con `userRating = 3`. Esto es consistente con el comportamiento de Games donde `userRating` puede ser null (no valorado) o un número (valorado).

### 9.2 Diferencia entre `notes` y `comment`

BoardGame ya tiene `notes: String`. Añadir `comment: String` puede parecer redundante. La distinción propuesta:

| Campo | Propósito |
|-------|-----------|
| `notes` | Notas técnicas: cuándo se jugó, con quién, variantes usadas, etc. (info factual) |
| `comment` | Opinión personal: qué te gustó, qué no, recomendación (valoración cualitativa) |

Si el usuario prefiere usar solo `notes` para todo, puede dejar `comment` vacío. No es obligatorio usar ambos.

**Alternativa:** renombrar `notes` a `comment` y eliminar `notes`. Pero eso es un cambio breaking (rompería datos existentes y APIs). Mejor añadir `comment` como campo nuevo y dejar `notes` como está.

### 9.3 Consistencia con Book.start (0-5 vs 1-5)

Book usa `start: Integer` con rango 0-5 donde 0 significa "sin valorar". Game usa `userRating: Integer` con rango 1-5 donde null significa "sin valorar". BoardGame propuesto usa `userRating: Integer 0-5` donde 0 = sin valorar, siguiendo el estilo de Book.

**Decisión:** usar 0-5 con 0 = sin valorar (como Book), no 1-5 con null (como Game). Razón: BoardGames son más similares a Books en el sentido de "colección poseída" que a Games (que tienen estados PLAYING/COMPLETED/ABANDONED). Además, permite distinguir explícitamente "no he valorado" (0) de "me gustó pero solo un poco" (1).

### 9.4 Visibilidad (Fase 9)

Si la colección de BoardGames es PÚBLICA, el rating personal y comentario son visibles para otros usuarios. Si es PRIVADA, solo el dueño los ve. El comportamiento es el mismo que `Game.userRating`/`Game.comment` en la Fase 9.

El `copyUpdatableFields` y los mappers no necesitan cambiar para soportar visibilidad — ya se filtra en el controller/service según la lógica de ownership de la Fase 9.

---

## 10. Esfuerzo estimado

| Área | Tiempo |
|-------|--------|
| Backend: modelo + entity + DTOs + mappers + service | 1-2h |
| Backend tests | 1h |
| Frontend: tipos + componentes (card, form, detail) | 2-3h |
| Frontend tests | 1h |
| **Total** | **5-7h** |

---

## 11. Documentos relacionados

- `docs/03-base-de-datos/03.1-tablas.md` — esquema actual de BoardGames
- `docs/05-api/README.md` — BoardGameRequest y BoardGameResponse actuales
- `docs/27-futuro-generos-books-games-boardgames.md` — añade `mechanics` a BoardGames (compatible con este cambio)
- `docs/24-futuro-reading-progress.md` — feature futura para Books

---

## 12. Orden de implementación

Este cambio es independiente de `27-futuro-generos-books-games-boardgames.md` (que añade `mechanics`). Se pueden hacer en paralelo o en cualquier orden. Si se hacen juntos, el cambio en `BoardGame` sería: añadir `mechanics + userRating + comment` de una vez.
