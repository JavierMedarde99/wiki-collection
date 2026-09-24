# Testing — Wiki-Collection

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

### Tests Implementados — Backend (68 archivos)

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
| CatboxClientTest | Unitario | Cliente Catbox con mock |
| HttpClientPropertiesTest | Unitario | Propiedades de timeout |
| WebConfigTest | Unitario | Configuración CORS |
| StringToEnumConvertersNullSafeTest | Unitario | Conversión null-safe |
| GlobalExceptionHandlerTest | Integración | Manejo global de excepciones |
| ImageStorageServiceTest | Unitario | Validación + upload imágenes |
| ImageStorageControllerTest | Integración | Endpoints de imágenes |
| UserPreferencesServiceTest | Unitario | CRUD de preferencias (Fase 9) |
| UserPreferencesControllerTest | Integración | Endpoints de preferencias (Fase 9) |
| UserProfileServiceTest | Unitario | Perfil público (Fase 9) |
| UserProfileControllerTest | Integración | Endpoints de perfil público (Fase 9) |
| BookServiceOwnerFilterTest | Unitario | Filtro owner=mine/other/all (Fase 9) |
| GameServiceOwnerFilterTest | Unitario | Filtro owner en juegos (Fase 9) |
| BoardGameServiceOwnerFilterTest | Unitario | Filtro owner en board games (Fase 9) |
| MagicCardServiceOwnerFilterTest | Unitario | Filtro owner en magic (Fase 9) |
| DeckServiceOwnerFilterTest | Unitario | Filtro owner en decks (Fase 9) |
| MovieShowServiceOwnerFilterTest | Unitario | Filtro owner en movieshows (Fase 9) |
| AuthServiceCreatePreferencesTest | Unitario | Crear preferencias al registrar (Fase 9) |
| AuthServiceTest | Unitario | Registro, login, refresh token |
| JwtServiceTest | Unitario | Generación y validación de tokens |
| AuthControllerTest | Integración | Endpoints de auth con MockMvc |
| JwtAuthenticationFilterTest | Unitario | Filtro con SecurityContext |
| SecurityConfigTest | Unitario | Configuración de seguridad |
| AuthDataMigrationTest | Integración | Creación de admin por defecto |
| UserDetailsServiceImplTest | Unitario | Carga de usuario para Spring Security |
| CurrentUserHandlerMethodArgumentResolverTest | Unitario | Resolver @CurrentUser |
| AuthJwtFlowTest | Integración | Flujo completo auth JWT |
| SecurityMatrixTest | Integración | Verificación de reglas de acceso |
| UserPreferencesMigrationTest | Integración | Migración de preferencias por defecto |
| UserOwnedBackfillMigrationTest | Integración | Poblar user_owned con datos existentes |
| OwnershipValidatorTest | Unitario | Validación de ownership |
| OwnerResolverTest | Unitario | Resolución de ownerId |
| OwnerScopeResolverTest | Unitario | Resolución de scope de visibilidad |
| StringToBoardGameStatusConverterTest | Unitario | Conversión String → BoardGameStatus |
| StringToBookStateConverterTest | Unitario | Conversión String → BookState |
| StringToMovieMediaTypeConverterTest | Unitario | Conversión String → MovieMediaType |
| StringToMovieStatusConverterTest | Unitario | Conversión String → MovieStatus |
| OpenApiConfigTest | Unitario | Configuración OpenAPI |
| StatsControllerTest | Integración | Endpoint de estadísticas globales |

**Total: 68 archivos de test backend**

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

### Tests Implementados — Frontend (10 archivos)

| Test | Tipo | Descripción |
|------|------|-------------|
| BookCard.test.tsx | Unitario | Componente BookCard |
| BookForm.test.tsx | Unitario | Componente BookForm |
| BookSearch.test.tsx | Unitario | Componente BookSearch |
| HomePage.test.tsx | Unitario | HomePage con stats |
| useInfiniteScroll.test.tsx | Unitario | Hook infinite scroll |
| PreferencesPage.test.tsx | Unitario | Página de preferencias (Fase 9) |
| CollectionPreferencesPanel.test.tsx | Unitario | Panel de preferencias (Fase 9) |
| PublicProfilePage.test.tsx | Unitario | Página de perfil público (Fase 9) |
| useCollectionPreferences.test.ts | Unitario | Hook de preferencias (Fase 9) |
| NavbarActiveCollections.test.tsx | Unitario | Navbar filtra por activas (Fase 9) |

**Total: 10 archivos de test frontend**

## Cobertura

| Capa | Cobertura Actual | Objetivo |
|------|-----------------|----------|
| Backend Service | ~80% | 90% |
| Backend Controller | ~70% | 85% |
| Frontend Components | ~25% | 75% |
| Frontend Pages | ~10% | 60% |

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
