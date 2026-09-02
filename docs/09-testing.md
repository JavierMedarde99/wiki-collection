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

## Frontend Tests

### Tests Unitarios (Vitest)

```javascript
import { render, screen } from "@testing-library/react";
import { test, expect } from "vitest";
import BookCard from "./BookCard";

test("renders book title", () => {
    const book = { title: "Test Book", author: "Author" };
    render(<BookCard book={book} />);
    expect(screen.getByText("Test Book")).toBeInTheDocument();
});
```

### Tests de Integración

```javascript
test("adds book to collection", async () => {
    render(<BookCreatePage />);
    fireEvent.change(screen.getByLabelText("Title"), {
        target: { value: "New Book" },
    });
    fireEvent.click(screen.getByText("Save"));
    expect(await screen.getByText("Book saved")).toBeInTheDocument();
});
```

## Cobertura

| Capa | Cobertura Actual | Objetivo |
|------|-----------------|----------|
| Backend Service | ~80% | 90% |
| Backend Controller | ~70% | 85% |
| Frontend Components | ~50% | 75% |
| Frontend Pages | ~30% | 60% |

## Comandos

```bash
# Backend
mvn test                    # Tests unitarios
mvn verify                  # Tests + integración

# Frontend
npm test                    # Tests unitarios
npm run test:coverage       # Cobertura
```
