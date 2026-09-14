1|# API Reference
2|
3|## Base URL
4|
5|```
6|http://localhost:8080/api/v1
7|```
8|
9|## Estructura
10|
11|| Sección | Descripción |
12||---------|-------------|
13|| [Endpoints de Libros](#endpoints-de-libros) | CRUD + búsqueda en Google Books |
14|| [Endpoints de Juegos](#endpoints-de-juegos) | CRUD + búsqueda en RAWG/FreeToGame + logros Steam |
15|| [Endpoints de Juegos de Mesa](#endpoints-de-juegos-de-mesa) | CRUD + búsqueda en BGG |
16|| [Endpoints de Magic](#endpoints-de-magic) | Listado, detalle, añadir desde Scryfall, eliminar + búsqueda |
17|| [Endpoints de Mazos](#endpoints-de-mazos) | CRUD + gestión cartas + status Commander |
18|| [Endpoints de Películas/Series](#endpoints-de-películas-series) | CRUD + búsqueda en TMDB |
19|| [Endpoints de Imágenes](#endpoints-de-imágenes) | Subida y eliminación de imágenes en Catbox |
20|| [APIs Externas](./externas/) | Integración con APIs externas |
21|
22|## APIs Externas
23|
24|| Archivo | Descripción | Estado |
25||---------|-------------|--------|
26|| [externas-books](./externas/externas-books.md) | Google Books API | ✅ Fase 1 |
27|| [externas-videogames](./externas/externas-videogames.md) | RAWG + FreeToGame | ✅ Fase 2 |
28|| [externas-steam](./externas/externas-steam.md) | Steam Web API (logros) | ✅ Fase 2 |
29|| [Guía: Steam API Key](./externas/steam-api-key-guide.md) | Cómo obtener tu API Key | ✅ Fase 2 |
30|| [externas-boardgames](./externas/externas-boardgames.md) | BoardGameGeek XML | ✅ Fase 3 |
31|| [externas-magic](./externas/externas-magic.md) | Scryfall | ✅ Fase 4 |
32|| [externas-movies](./externas/externas-movies.md) | TMDB | ✅ Fase 5 |
33|| [Image Hosting](./externas/externas-image-hosting.md) | Catbox.moe para subir imágenes | ✅ Fase 6 |
34|
35|## Códigos de Estado
36|
37|| Código | Significado |
38||--------|-------------|
39|| 200 | OK |
40|| 201 | Created |
41|| 204 | No Content |
42|| 400 | Bad Request |
43|| 404 | Not Found |
44|| 409 | Conflicto (duplicado) |
45|| 500 | Server Error |
46|| 502 | Bad Gateway (API externa) |
47|
48|## Paginación
49|
50|Los endpoints que devuelven listas soportan paginación Spring Data:
51|
52|```
53|GET /api/v1/books?page=0&size=20&sort=title,asc
54|GET /api/v1/games?page=0&size=20&sort=title,asc
55|GET /api/v1/boardgames?page=0&size=20&sort=title,asc
56|GET /api/v1/magic?page=0&size=20&sort=name,asc
57|GET /api/v1/movieshows?page=0&size=20&sort=title,asc
58|```
59|
60|**Response:**
61|```json
62|{
63|  "content": [...],
64|  "totalPages": 5,
65|  "totalElements": 100,
66|  "number": 0,
67|  "size": 20
68|}
69|```
70|
71|---
72|
73|## Endpoints de Libros
74|
75|| Método | Endpoint | Descripción | Estado |
76||--------|----------|-------------|--------|
77|| GET | `/api/v1/books` | Listar libros con paginación y filtros | ✅ |
78|| GET | `/api/v1/books/{id}` | Obtener libro por ID | ✅ |
79|| POST | `/api/v1/books` | Crear libro | ✅ |
80|| PUT | `/api/v1/books/{id}` | Actualizar libro | ✅ |
81|| DELETE | `/api/v1/books/{id}` | Eliminar libro (204 No Content) | ✅ |
82|| GET | `/api/v1/books/search?name={query}` | Buscar en Google Books API | ✅ |
83|
84|### Filtros de Libros
85|
86|| Parámetro | Tipo | Descripción | Ejemplo |
87||-----------|------|-------------|---------|
88|| `name` | String | Buscar por título (LIKE case-insensitive) | `?name=harry` |
89|| `author` | String | Buscar por autor (LIKE case-insensitive) | `?author=rowling` |
90|| `type` | Enum | Filtrar por tipo (MANGA, NOVEL, GRAPHIC_NOVEL) | `?type=MANGA` |
91|| `state` | Enum | Filtrar por estado (TO_READ, READING, COMPLETED) | `?state=READING` |
92|
93|---
94|
95|## Endpoints de Juegos
96|
97|| Método | Endpoint | Descripción | Estado |
98||--------|----------|-------------|--------|
99|| GET | `/api/v1/games` | Listar juegos con paginación y filtros | ✅ |
100|| GET | `/api/v1/games/{id}` | Obtener juego por ID | ✅ |
101|| POST | `/api/v1/games` | Crear juego | ✅ |
102|| PUT | `/api/v1/games/{id}` | Actualizar juego | ✅ |
103|| DELETE | `/api/v1/games/{id}` | Eliminar juego (204 No Content) | ✅ |
104|| GET | `/api/v1/games/search?name={query}` | Buscar en RAWG (fallback a FreeToGame) | ✅ |
105|| GET | `/api/v1/games/{gameId}/achievements?steamId={id}` | Logros de un jugador (Steam) | ✅ |
106|
107|### Filtros de Juegos
108|
109|| Parámetro | Tipo | Descripción | Ejemplo |
110||-----------|------|-------------|---------|
111|| `name` | String | Buscar por título | `?name=witcher` |
112|| `platform` | Enum | Filtrar por plataforma (PC, PS2, PS3, WII_U, SWITCH) | `?platform=PC` |
113|| `status` | Enum | Filtrar por estado (PLAYING, COMPLETED, WISHLIST, ABANDONED) | `?status=PLAYING` |
114|
115|---
116|
117|## Endpoints de Juegos de Mesa
118|
119|| Método | Endpoint | Descripción | Estado |
120||--------|----------|-------------|--------|
121|| GET | `/api/v1/boardgames` | Listar juegos de mesa con paginación y filtros | ✅ |
122|| GET | `/api/v1/boardgames/{id}` | Obtener juego de mesa por ID | ✅ |
123|| POST | `/api/v1/boardgames` | Crear juego de mesa | ✅ |
124|| PUT | `/api/v1/boardgames/{id}` | Actualizar juego de mesa | ✅ |
125|| DELETE | `/api/v1/boardgames/{id}` | Eliminar juego de mesa (204 No Content) | ✅ |
126|| GET | `/api/v1/boardgames/search?name={query}` | Buscar en BoardGameGeek (XML API) | ✅ |
127|
128|### Filtros de Juegos de Mesa
129|
130|| Parámetro | Tipo | Descripción | Ejemplo |
131||-----------|------|-------------|---------|
132|| `name` | String | Buscar por título | `?name=catan` |
133|| `status` | Enum | Filtrar por estado (OWNED, WISHLIST) | `?status=OWNED` |
134|
135|---
136|
137|## Endpoints de Magic: The Gathering
138|
139|| Método | Endpoint | Descripción | Estado |
140||--------|----------|-------------|--------|
141|| GET | `/api/v1/magic` | Listar cartas con paginación y filtros | ✅ |
142|| GET | `/api/v1/magic/{id}` | Obtener carta por ID | ✅ |
143|| POST | `/api/v1/magic/scryfall/{scryfallId}` | Añadir carta desde Scryfall | ✅ |
144|| DELETE | `/api/v1/magic/{id}` | Eliminar carta (204 No Content) | ✅ |
145|| GET | `/api/v1/magic/search?name={query}` | Buscar en Scryfall | ✅ |
146|| GET | `/api/v1/magic/commanders?colors={colors}` | Buscar comandantes por colores | ✅ |
147|
148|**Nota:** El backend NO expone POST/PUT genéricos para Magic. Las cartas solo se pueden añadir desde Scryfall con `POST /scryfall/{scryfallId}`.
149|
150|### Filtros de Magic
151|
152|| Parámetro | Tipo | Descripción | Ejemplo |
153||-----------|------|-------------|---------|
154|| `name` | String | Buscar por nombre (LIKE case-insensitive) | `?name=lightning` |
155|| `rarity` | String | Filtrar por rareza | `?rarity=rare` |
156|| `color` | String | Filtrar por color | `?color=R` |
157|| `type` | String | Filtrar por tipo | `?type=Creature` |
158|
159|---
160|
161|## Endpoints de Mazos (Commander)
162|
163|| Método | Endpoint | Descripción | Estado |
164||--------|----------|-------------|--------|
165|| GET | `/api/v1/decks` | Listar mazos (filtro por nombre opcional) | ✅ |
166|| GET | `/api/v1/decks/{id}` | Obtener mazo por ID | ✅ |
167|| POST | `/api/v1/decks` | Crear mazo | ✅ |
168|| PUT | `/api/v1/decks/{id}` | Actualizar mazo | ✅ |
169|| DELETE | `/api/v1/decks/{id}` | Eliminar mazo (204 No Content) | ✅ |
170|| POST | `/api/v1/decks/{id}/cards` | Añadir carta desde Scryfall | ✅ |
171|| DELETE | `/api/v1/decks/{id}/cards/{scryfallId}` | Quitar carta del mazo | ✅ |
172|| GET | `/api/v1/decks/{id}/status` | Estado del mazo (DRAFT, COMPLETE, INVALID) | ✅ |
173|
174|### Filtros de Mazos
175|
176|| Parámetro | Tipo | Descripción | Ejemplo |
177||-----------|------|-------------|---------|
178|| `name` | String | Filtrar por nombre exacto | `?name=Mi Mazo` |
179|
180|### DeckCardRequest (body para añadir carta)
181|
182|```json
183|{
184|  "scryfallId": "abc123",
185|  "quantity": 1
186|}
187|```
188|
189|### DeckStatusResponse
190|
191|```json
192|{
193|  "status": "COMPLETE",
194|  "message": "El mazo cumple las reglas Commander"
195|}
196|```
197|
198|---
199|
200|## Endpoints de Películas/Series
201|
202|| Método | Endpoint | Descripción | Estado |
203||--------|----------|-------------|--------|
204|| GET | `/api/v1/movieshows` | Listar películas/series con paginación y filtros | ✅ |
205|| GET | `/api/v1/movieshows/{id}` | Obtener película/serie por ID | ✅ |
206|| POST | `/api/v1/movieshows` | Crear película/serie | ✅ |
207|| PUT | `/api/v1/movieshows/{id}` | Actualizar película/serie | ✅ |
208|| DELETE | `/api/v1/movieshows/{id}` | Eliminar película/serie (204 No Content) | ✅ |
209|| GET | `/api/v1/movieshows/search?name={query}&mediaType={type}` | Buscar en TMDB | ✅ |
210|
211|### Filtros de Películas/Series
212|
213|| Parámetro | Tipo | Descripción | Ejemplo |
214||-----------|------|-------------|---------|
215|| `name` | String | Buscar por título | `?name=matrix` |
216|| `status` | Enum | Filtrar por estado (WATCHING, WATCHED, PLAN_TO_WATCH) | `?status=WATCHING` |
217|| `mediaType` | Enum | Filtrar por tipo (MOVIE, TV) | `?mediaType=MOVIE` |
218|
219|---
220|
221|## Endpoints de Imágenes
222|
223|| Método | Endpoint | Descripción | Estado |
224||--------|----------|-------------|--------|
225|| POST | `/api/v1/images/upload` | Subir imagen a Catbox (multipart, campo `file`) → 201 `{url, filename}` | ✅ |
| DELETE | `/api/v1/images/{filename}` | Eliminar en Catbox (204; requiere userhash) | ✅ |

### Upload Image

```http
POST /api/v1/images/upload
Content-Type: multipart/form-data

file: <archivo>
```

**Response:**
```json
{
  "url": "https://files.catbox.moe/abc123.jpg",
  "filename": "abc123.jpg"
}
```

**Flujo:** `POST /api/v1/images/upload` → Catbox devuelve `url` → usarla en el campo de imagen de cada entidad (`frontpage`, `thumbnailUrl`, `posterUrl`, etc.).

```bash
curl -X POST -F "file=@foto.jpg" http://localhost:8080/api/v1/images/upload
# {"url":"https://files.catbox.moe/abc123.jpg","filename":"abc123.jpg"}
```

**Validaciones:** 5 MB máximo, MIME permitido (JPEG, PNG, GIF, WebP) con comprobación de magic bytes.

**Configuración:** `catbox.api.base-url`, `catbox.userhash` (necesario solo para borrar).

254|```json
255|{
256|  "url": "https://files.catbox.moe/abc123.jpg",
257|  "filename": "abc123.jpg"
258|}
259|```
260|
261|**Validaciones:**
262|- Tamaño máximo: 5 MB
263|- MIME permitido: image/jpeg, image/png, image/gif, image/webp
264|- Verificación de magic bytes
265|
266|---
267|
268|## DTOs de Request/Response
269|
270|### BookRequest
271|```json
272|{
273|  "externalId": "string",
274|  "title": "string (obligatorio)",
275|  "descripcion": "string",
276|  "author": "string (obligatorio)",
277|  "pages": "integer (min 0)",
278|  "type": "MANGA | NOVEL | GRAPHIC_NOVEL",
279|  "state": "TO_READ | READING | COMPLETED",
280|  "comment": "string",
281|  "start": "integer (0-5)",
282|  "startDate": "date",
283|  "endDate": "date",
284|  "frontpage": "string (URL)"
285|}
286|```
287|
288|### GameRequest
289|```json
290|{
291|  "externalId": "string",
292|  "title": "string (obligatorio)",
293|  "platform": "PC | PS2 | PS3 | WII_U | SWITCH",
294|  "thumbnailUrl": "string",
295|  "status": "PLAYING | COMPLETED | WISHLIST | ABANDONED",
296|  "userRating": "integer (1-5)",
297|  "comment": "string",
298|  "dateAdded": "date",
299|  "dateCompleted": "date",
300|  "externalSource": "string",
301|  "steamAppId": "string",
302|  "obtainPlatinum": "boolean"
303|}
304|```
305|
306|### BoardGameRequest
307|```json
308|{
309|  "title": "string (obligatorio)",
310|  "description": "string",
311|  "yearPublished": "integer",
312|  "minPlayers": "integer (min 1)",
313|  "maxPlayers": "integer (min 1)",
314|  "minPlaytime": "integer (min 1)",
315|  "maxPlaytime": "integer (min 1)",
316|  "publisher": "string",
317|  "designers": ["string"],
318|  "categories": ["string"],
319|  "mechanics": ["string"],
320|  "imageUrl": "string",
321|  "thumbnailUrl": "string",
322|  "bggRating": "number (0-10)",
323|  "bggId": "string",
324|  "status": "OWNED | WISHLIST",
325|  "notes": "string",
326|  "dateAdded": "date"
327|}
328|```
329|
330|### MovieShowRequest
331|```json
332|{
333|  "externalId": "string (obligatorio)",
334|  "title": "string (obligatorio)",
335|  "overview": "string",
336|  "releaseDate": "date",
337|  "posterUrl": "string",
338|  "backdropUrl": "string",
339|  "voteAverage": "number",
340|  "mediaType": "MOVIE | TV",
341|  "status": "WATCHING | WATCHED | PLAN_TO_WATCH",
342|  "userRating": "integer (1-5)",
343|  "comment": "string",
344|  "dateAdded": "date",
345|  "dateCompleted": "date",
346|  "externalSource": "string"
347|}
348|```
349|
350|### DeckRequest
351|```json
352|{
353|  "name": "string (obligatorio)",
354|  "description": "string",
355|  "commander": "string",
356|  "commanderColors": ["string"]
357|}
358|```
359|
360|### DeckCardRequest
361|```json
362|{
363|  "scryfallId": "string (obligatorio)",
364|  "quantity": "integer (min 1)"
365|}
366|```
367|
368|### AchievementsResponse
369|```json
370|{
371|  "achievements": [
372|    {
373|      "name": "string",
374|      "description": "string",
375|      "achieved": "boolean",
376|      "iconUrl": "string"
377|    }
378|  ],
379|  "totalAchievements": "integer",
380|  "totalAchieved": "integer",
381|  "percentage": "number"
382|}
383|```
384|
385|### ImageResponse
386|```json
387|{
388|  "url": "https://files.catbox.moe/abc123.jpg",
389|  "filename": "abc123.jpg"
390|}
391|```
392|
393|### ErrorResponse
394|```json
395|{
396|  "timestamp": "2024-01-01T12:00:00",
397|  "status": 404,
398|  "error": "Not Found",
399|  "message": "Libro no encontrado",
400|  "path": "/api/v1/books/123"
401|}
402|```
403|