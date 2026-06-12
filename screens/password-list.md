# PasswordListScreen

## Descripción

Pantalla principal que muestra la lista de contraseñas del usuario. Incluye búsqueda en tiempo real, filtrado por categoría, y organización por secciones (favoritos y otras).

**Archivo**: `PasswordListScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  🔍 Buscar por título, usuario, URL...  │
├─────────────────────────────────────────┤
│  [Todas] [General] [Redes] [Finanzas]   │
├─────────────────────────────────────────┤
│                                         │
│  FAVORITOS                              │
│  ┌─────────────────────────────────┐   │
│  │ 🔒 Google           👤 user  ⭐  │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │ 🔒 Facebook         👤 user  ⭐  │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ─────────────────────────────────────  │
│                                         │
│  OTRAS CONTRASEÑAS                      │
│  ┌─────────────────────────────────┐   │
│  │ 🔒 Amazon           👤 user  ☆  │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │ 🔒 Netflix          👤 user  ☆  │   │
│  └─────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
                              ┌────┐
                              │ +  │  FAB
                              └────┘
```

## Componentes

### SearchBar

**Ubicación**: Parte superior, siempre visible

**Propiedades**:
- Placeholder: "Buscar por título, usuario o URL..."
- Icono: `Icons.Default.Search`
- Botón de limpiar cuando hay texto

**Comportamiento**:
- Filtra en tiempo real mientras el usuario escribe
- Búsqueda case-insensitive
- Busca en: título, username, URL

### CategoryFilterChips

**Ubicación**: Debajo del SearchBar

**Componentes**:
- Chip "Todas" (resetea filtros)
- Chips de categorías (scroll horizontal)

**Estados**:
- Seleccionado: Color de la categoría
- No seleccionado: Gris por defecto

### Lista de Contraseñas

**Organización**:
1. **Sección Favoritos**: Solo si hay entradas favoritas
2. **Separador**: Línea horizontal divisoria
3. **Sección Otras**: Resto de contraseñas

**Empty States**:
- Sin contraseñas: "No hay contraseñas guardadas. ¡Crea una nueva para comenzar!"
- Sin resultados: "No se encontraron contraseñas con los filtros actuales"

## Estado (State)

```kotlin
data class PasswordListState(
    val entries: List<PasswordEntry> = emptyList(),
    val searchQuery: String = "",
    val selectedCategoryId: String? = null,
    val isLoading: Boolean = true,
    val error: String? = null
)
```

| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `entries` | `List<PasswordEntry>` | Lista filtrada de contraseñas |
| `searchQuery` | `String` | Texto actual de búsqueda |
| `selectedCategoryId` | `String?` | ID de categoría seleccionada (null = todas) |
| `isLoading` | `Boolean` | true mientras se cargan datos |
| `error` | `String?` | Mensaje de error si ocurre |

## Eventos de UI

| Evento | Parámetros | Descripción |
|--------|------------|-------------|
| `onSearchQueryChanged` | `query: String` | Usuario escribe en búsqueda |
| `onCategoryFilterSelected` | `categoryId: String?` | Usuario selecciona categoría |
| `clearCategoryFilter` | - | Usuario pulsa "Todas" |
| `onToggleFavorite` | `entry: PasswordEntry` | Usuario pulsa estrella |
| `onNavigateToDetail` | `id: String` | Usuario pulsa entrada |
| `onDeleteEntry` | `entry: PasswordEntry` | Usuario confirma eliminar |

## Comportamiento de la Lista

### Filtrado

El filtrado se realiza combinando:
1. **Búsqueda de texto**: `title`, `username`, `url`
2. **Categoría seleccionada**: `categoryId`

```kotlin
val filtered = entries.filter { entry ->
    val matchesSearch = searchQuery.isBlank() ||
        entry.title.contains(searchQuery, ignoreCase = true) ||
        entry.username.contains(searchQuery, ignoreCase = true) ||
        entry.url?.contains(searchQuery, ignoreCase = true) == true
    
    val matchesCategory = selectedCategoryId == null || 
        entry.categoryId == selectedCategoryId
    
    matchesSearch && matchesCategory
}
```

### Ordenamiento

Las entradas se ordenan alfabéticamente por título dentro de cada sección.

### Sección de Favoritos

```kotlin
val favoriteEntries = entries.filter { it.isFavorite }
val otherEntries = entries.filter { !it.isFavorite }
```

**Renderizado**:
- Si `favoriteEntries.isNotEmpty()`: Mostrar sección con header "Favoritos"
- Separador horizontal
- Header "Otras contraseñas"
- Lista de `otherEntries`

## PasswordEntryItem

Cada entrada en la lista se muestra con:

```kotlin
Card(
    modifier = Modifier
        .fillMaxWidth()
        .clickable { onNavigateToDetail(entry.id) }
) {
    Row {
        // Icono (candado)
        Icon(Icons.Default.Lock, tint = primary)
        
        // Información
        Column {
            Text(entry.title)           // Título, ellipsis si es largo
            Text(entry.username)        // Username pequeño
            // URL si existe
        }
        
        // Botón favorito
        IconButton({ onToggleFavorite(entry) }) {
            Icon(
                if (entry.isFavorite) Icons.Filled.Star 
                else Icons.Outlined.Star,
                tint = if (entry.isFavorite) Color(0xFFFFD700) else onSurfaceVariant
            )
        }
        
        // Menú contextual (tres puntos)
        IconButton({ showContextMenu = true }) {
            Icon(Icons.Default.MoreVert, contentDescription = "Más opciones")
        }
    }
    
    // DropdownMenu con Editar/Eliminar
}
```

## Menú Contextual

Al pulsar los tres puntos:

```
┌─────────────────┐
│ ✏️  Editar     │
│ 🗑️  Eliminar   │
└─────────────────┘
```

**Acciones**:
- **Editar**: Navega a `PasswordFormScreen` con el ID de la entrada
- **Eliminar**: Muestra diálogo de confirmación

## Diálogo de Eliminación

```kotlin
AlertDialog(
    icon = { Icon(Icons.Default.Delete, tint = error) },
    title = { Text("¿Eliminar contraseña?") },
    text = { Text("Esta acción no se puede deshacer") },
    dismissButton = {
        TextButton(onClick = { showDeleteDialog = false }) {
            Text("Cancelar")
        }
    },
    confirmButton = {
        TextButton(onClick = {
            viewModel.onDeleteEntry(entry)
            showDeleteDialog = false
        }) {
            Text("Eliminar", color = error)
        }
    }
)
```

## Floating Action Button

**Ubicación**: Esquina inferior derecha

**Acción**: Navegar a `PasswordFormScreen` para crear nueva contraseña

```kotlin
FloatingActionButton(
    onClick = onNavigateToCreate,
    modifier = Modifier
        .align(Alignment.BottomEnd)
        .padding(16.dp)
) {
    Icon(Icons.Default.Add, contentDescription = "Añadir contraseña")
}
```

## Navegación

| Acción | Destino | Parámetros |
|--------|---------|------------|
| Crear nueva | `PasswordFormScreen` | - |
| Ver detalle | `PasswordDetailScreen` | `id: String` |
| Editar | `PasswordFormScreen` | `id: String` |

## Flujo de Datos

```
Usuario escribe en SearchBar
         ↓
PasswordListViewModel.onSearchQueryChanged(query)
         ↓
Actualiza _searchQuery.value
         ↓
observeFilters() combina flows
         ↓
Filtra entries en memoria
         ↓
Actualiza _state.value.entries
         ↓
UI se recompone con nueva lista
```

## Casos de Uso Involucrados

| Caso de Uso | Propósito |
|-------------|-----------|
| `GetAllPasswords` | Obtener todas las contraseñas |
| `DeletePasswordEntry` | Eliminar una contraseña |
| `ToggleFavoriteEntry` | Marcar/desmarcar favorito |

## Testing

### UI Tests

```kotlin
@Test
fun passwordListScreen_displaysFavoritesSection() {
    // Given
    val favorites = listOf(
        PasswordEntry(id = "1", title = "Google", isFavorite = true, /* ... */)
    )
    val others = listOf(
        PasswordEntry(id = "2", title = "Amazon", isFavorite = false, /* ... */)
    )
    
    // When
    composeTestRule.setContent {
        PasswordListScreen(viewModel = fakeViewModel(favorites + others))
    }
    
    // Then
    composeTestRule.onNodeWithText("Favoritos").assertIsDisplayed()
    composeTestRule.onNodeWithText("Otras contraseñas").assertIsDisplayed()
    composeTestRule.onNodeWithText("Google").assertIsDisplayed()
    composeTestRule.onNodeWithText("Amazon").assertIsDisplayed()
}

@Test
fun passwordListScreen_filterByCategory_showsOnlyMatching() {
    // Given
    val entries = listOf(
        PasswordEntry(id = "1", title = "Google", categoryId = "cat-1"),
        PasswordEntry(id = "2", title = "Facebook", categoryId = "cat-2")
    )
    
    // When
    composeTestRule.setContent {
        PasswordListScreen(
            viewModel = fakeViewModel(entries),
            categories = listOf(Category(id = "cat-1", name = "General"))
        )
    }
    
    composeTestRule.onNodeWithText("cat-1").performClick()
    
    // Then
    composeTestRule.onNodeWithText("Google").assertIsDisplayed()
    composeTestRule.onNodeWithText("Facebook").assertDoesNotExist()
}
```

### ViewModel Tests

```kotlin
@Test
fun `should filter entries by search query`() = runTest {
    // Given
    val entries = listOf(
        PasswordEntry(title = "Google", username = "user@gmail.com"),
        PasswordEntry(title = "Facebook", username = "user@fb.com")
    )
    coEvery { getAllPasswords() } returns flowOf(entries)
    
    val viewModel = PasswordListViewModel(getAllPasswords, ...)
    
    // When
    viewModel.onSearchQueryChanged("gmail")
    
    // Then
    val state = viewModel.state.first()
    assertEquals(1, state.entries.size)
    assertEquals("Google", state.entries.first().title)
}

@Test
fun `should toggle favorite when clicked`() = runTest {
    // Given
    val entry = PasswordEntry(id = "1", isFavorite = false)
    coEvery { toggleFavoriteEntry(entry) } returns Result.success(Unit)
    
    val viewModel = PasswordListViewModel(...)
    
    // When
    viewModel.onToggleFavorite(entry)
    
    // Then
    coVerify { toggleFavoriteEntry(entry) }
}
```

## Accesibilidad

- **ContentDescription**: Cada icono tiene descripción
- **TouchTarget**: Mínimo 48dp para botones
- **TalkBack**: Anuncia "Favorito" para entradas marcadas
- **Keyboard**: Navegación con teclado soportada

## Rendimiento

### Optimizaciones

1. **LazyColumn**: Solo renderiza elementos visibles
2. **keys únicas**: `key = { it.id }` para reciclado eficiente
3. **Flow + combine**: Filtrado reactivo sin polling
4. **StateFlow**: Solo emite cuando hay cambios reales

### Consideraciones

Para listas grandes (>1000 elementos):
- Considerar paginación con Paging 3
- Implementar búsqueda en BD en lugar de memoria

## Referencias

- [LazyColumn Documentation](https://developer.android.com/jetpack/compose/lists)
- [Flow Documentation](https://kotlinlang.org/docs/flow.html)
- [Material Design Lists](https://m3.material.io/components/lists)

---

**Documentación Relacionada:**
- [PasswordDetailScreen](password-detail.md)
- [PasswordFormScreen](password-form.md)
- [CategoryFilterChips](../components/category-filter-chips.md)
