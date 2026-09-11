# Testing

## Backend Tests

### Tests Unitarios

```java
@ExtendWith(MockitoExtension.class)
class BookServiceTest {
    @Mock private BookRepository repository;
    @InjectMocks private BookService service;
    
    @Test
    void shouldSaveBook() {
        Book book = new Book("Title", "Author", ...);
        when(repository.save(any())).thenReturn(book);
        
        Book saved = service.save(book);
        
        assertThat(saved.title()).isEqualTo("Title");
        verify(repository).save(book);
    }
}
```

### Tests de Integración

```java
@WebMvcTest(BookController.class)
class BookControllerTest {
    @Autowired private MockMvc mockMvc;
    @MockBean private BookUseCase useCase;
    
    @Test
    void shouldReturnBooks() throws Exception {
        when(useCase.findAll(any())).thenReturn(Page.empty());
        
        mockMvc.perform(get("/api/books"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.content").isArray());
    }
}
```

### Tests con Mock Server

```java
@ExtendWith(MockitoExtension.class)
class GoogleBooksClientTest {
    @Test
    void shouldSearchBooks() {
        // Usa MockWebServer para simular Google Books API
    }
}
```

### Tests Implementados

| Test | Tipo | Descripción |
|------|------|-------------|
| BookServiceTest | Unitario | CRUD de libros |
| BookControllerTest | Integración | Endpoints de libros |
| BookSearchServiceTest | Unitario | Búsqueda en Google Books |
| GoogleBooksClientTest | Unitario | Cliente Google Books con mock |
| BookPersistenceAdapterTest | Integración | Persistencia de libros |
| GameServiceTest | Unitario | CRUD de juegos |
| GameControllerTest | Integración | Endpoints de juegos |
| GameSearchServiceTest | Unitario | Búsqueda con fallback |
| GameAchievementsServiceTest | Unitario | Logros de Steam |
| RAWGClientTest | Unitario | Cliente RAWG con mock |
| FreeToGameClientTest | Unitario | Cliente FreeToGame con mock |
| GamePersistenceAdapterTest | Integración | Persistencia de juegos |
| GameDtoMapperTest | Unitario | Mapeo DTO ↔ Domain |
| BoardGameServiceTest | Unitario | CRUD de juegos de mesa |
| BoardGameControllerTest | Integración | Endpoints de juegos de mesa |
| BoardGameSearchServiceTest | Unitario | Búsqueda BGG |
| BggXmlClientTest | Unitario | Cliente BGG XML con mock |
| BoardGamePersistenceAdapterTest | Integración | Persistencia de juegos de mesa |
| BoardGameXmlMapperTest | Unitario | Mapeo XML → BoardGame |
| BoardGameStatusMigrationTest | Unitario | Migración de estado |
| MagicCardServiceTest | Unitario | Listado/eliminación de cartas Magic |
| MagicCardControllerTest | Integración | Endpoints de Magic |
| MagicCardSearchServiceTest | Unitario | Búsqueda en Scryfall |
| ScryfallClientTest | Unitario | Cliente Scryfall con mock |
| MagicCardPersistenceAdapterTest | Integración | Persistencia de Magic |
| MagicCardMapperTest | Unitario | Mapeo JSON → MagicCard |
| MagicCardDtoMapperTest | Unitario | Mapeo DTO ↔ Domain |
| DateRangeValidatorTest | Unitario | Validación de fechas |
| StringToGameStatusConverterTest | Unitario | Conversión de estado |
| GameTest | Unitario | Modelo de dominio Game |
| MovieShowServiceTest | Unitario | CRUD de películas/series |
| MovieShowControllerTest | Integración | Endpoints de películas/series |
| MovieSearchServiceTest | Unitario | Búsqueda en TMDB |
| TmdbClientTest | Unitario | Cliente TMDB con mock |
| MovieShowPersistenceAdapterTest | Integración | Persistencia de películas/series |
| MovieShowDtoMapperTest | Unitario | Mapeo DTO ↔ Domain |
| DeckServiceTest | Unitario | CRUD de mazos |
| DeckControllerTest | Integración | Endpoints de mazos |
| DeckSearchServiceTest | Unitario | Búsqueda de comandantes |
| DeckValidationTest | Unitario | Validación de mazos |
| DeckPersistenceAdapterTest | Integración | Persistencia de mazos |
| DeckDtoMapperTest | Unitario | Mapeo DTO ↔ Domain |
| DeckModelTest | Unitario | Modelo de dominio Deck |
| MovieShowModelTest | Unitario | Modelo de dominio MovieShow |

## Frontend Tests

### Tests Unitarios (Vitest)

```typescript
import { render, screen } from "@testing-library/react";
import { test, expect } from "vitest";
import BookCard from "./BookCard";

test("renders book title", () => {
    const book = { title: "Test Book", author: "Author" };
    render(<BookCard book={book} />);
    expect(screen.getByText("Test Book")).toBeInTheDocument();
});
```

### Tests Implementados

| Test | Tipo | Descripción |
|------|------|-------------|
| BookCard.test.tsx | Unitario | Componente BookCard |
| BookForm.test.tsx | Unitario | Componente BookForm |
| BookSearch.test.tsx | Unitario | Componente BookSearch |

## Cobertura

| Capa | Cobertura Actual | Objetivo |
|------|-----------------|----------|
| Backend Service | ~80% | 90% |
| Backend Controller | ~70% | 85% |
| Frontend Components | ~50% | 75% |
| Frontend Pages | ~30% | 60% |

**Nota:** Jacoco está configurado con un umbral mínimo de 80% de cobertura de línea. El build falla si no se alcanza.

## Comandos

```bash
# Backend
mvn test                    # Tests unitarios
mvn verify                  # Tests + integración + Jacoco

# Frontend
npm test                    # Tests unitarios (Vitest)
npm run test:coverage       # Cobertura
```
