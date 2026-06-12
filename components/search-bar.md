# SearchBar

## Visión General

Componente reutilizable de barra de búsqueda con icono y botón de limpiar.

**Archivo**: `presentation/ui/components/SearchBar.kt`

## Uso Básico

```kotlin
@Composable
fun PasswordListScreen(...) {
    var searchQuery by remember { mutableStateOf("") }
    
    SearchBar(
        query = searchQuery,
        onQueryChanged = { searchQuery = it },
        placeholder = "Buscar contraseñas..."
    )
}
```

## API del Componente

```kotlin
@Composable
fun SearchBar(
    query: String,                              // Texto actual de búsqueda
    onQueryChanged: (String) -> Unit,           // Callback al cambiar
    placeholder: String = "Buscar...",          // Placeholder
    modifier: Modifier = Modifier               // Modificador opcional
)
```

## Implementación

```kotlin
@Composable
fun SearchBar(
    query: String,
    onQueryChanged: (String) -> Unit,
    placeholder: String = "Buscar...",
    modifier: Modifier = Modifier
) {
    OutlinedTextField(
        value = query,
        onValueChange = onQueryChanged,
        placeholder = { Text(placeholder) },
        leadingIcon = {
            Icon(
                imageVector = Icons.Default.Search,
                contentDescription = null,
                tint = MaterialTheme.colorScheme.onSurfaceVariant
            )
        },
        trailingIcon = {
            if (query.isNotEmpty()) {
                IconButton(onClick = { onQueryChanged("") }) {
                    Icon(
                        imageVector = Icons.Default.Close,
                        contentDescription = "Limpiar búsqueda",
                        tint = MaterialTheme.colorScheme.onSurfaceVariant
                    )
                }
            }
        },
        singleLine = true,
        colors = OutlinedTextFieldDefaults.colors(
            focusedBorderColor = MaterialTheme.colorScheme.primary,
            unfocusedBorderColor = MaterialTheme.colorScheme.outline
        ),
        modifier = modifier.fillMaxWidth()
    )
}
```

## Características

| Característica | Descripción |
|---------------|-------------|
| Icono de búsqueda | Siempre visible a la izquierda |
| Botón de limpiar | Solo aparece cuando hay texto |
| Single line | Una sola línea de texto |
| Bordes temáticos | Primary cuando enfocado, Outline cuando no |
| Full width | Ocupa todo el ancho disponible por defecto |

## Variantes

### Con botón de acción adicional

```kotlin
@Composable
fun SearchBarWithFilter(
    query: String,
    onQueryChanged: (String) -> Unit,
    onFilterClick: () -> Unit,
    modifier: Modifier = Modifier
) {
    OutlinedTextField(
        value = query,
        onValueChange = onQueryChanged,
        placeholder = { Text("Buscar...") },
        leadingIcon = {
            Icon(Icons.Default.Search, null)
        },
        trailingIcon = {
            Row {
                if (query.isNotEmpty()) {
                    IconButton(onClick = { onQueryChanged("") }) {
                        Icon(Icons.Default.Close, null)
                    }
                }
                IconButton(onClick = onFilterClick) {
                    Icon(Icons.Default.FilterList, null)
                }
            }
        },
        modifier = modifier.fillMaxWidth()
    )
}
```

### Con estado de carga

```kotlin
@Composable
fun SearchBarWithLoading(
    query: String,
    onQueryChanged: (String) -> Unit,
    isLoading: Boolean,
    modifier: Modifier = Modifier
) {
    OutlinedTextField(
        value = query,
        onValueChange = onQueryChanged,
        placeholder = { Text("Buscar...") },
        leadingIcon = {
            if (isLoading) {
                CircularProgressIndicator(
                    modifier = Modifier.size(20.dp),
                    strokeWidth = 2.dp
                )
            } else {
                Icon(Icons.Default.Search, null)
            }
        },
        trailingIcon = {
            if (query.isNotEmpty() && !isLoading) {
                IconButton(onClick = { onQueryChanged("") }) {
                    Icon(Icons.Default.Close, null)
                }
            }
        },
        modifier = modifier.fillMaxWidth()
    )
}
```

## Accesibilidad

```kotlin
SearchBar(
    query = searchQuery,
    onQueryChanged = { searchQuery = it },
    placeholder = "Buscar contraseñas",
    modifier = Modifier
        .fillMaxWidth()
        .semantics {
            label = "Barra de búsqueda"
            placeholderText = "Buscar contraseñas"
        }
)
```

## Testing

```kotlin
@Test
fun searchBar_clearButton_visibleWhenQueryNotEmpty() {
    composeTestRule.setContent {
        SearchBar(
            query = "test",
            onQueryChanged = {}
        )
    }
    
    // El botón de limpiar debe ser visible
    composeTestRule
        .onNodeWithContentDescription("Limpiar búsqueda")
        .assertExists()
        .assertIsEnabled()
}

@Test
fun searchBar_clearButton_notVisibleWhenEmpty() {
    composeTestRule.setContent {
        SearchBar(
            query = "",
            onQueryChanged = {}
        )
    }
    
    // El botón de limpiar no debe existir
    composeTestRule
        .onNodeWithContentDescription("Limpiar búsqueda")
        .assertDoesNotExist()
}
```

## Referencias

- [OutlinedTextField - Material 3](https://m3.material.io/components/text-fields)
- [Icons - Material Icons](https://fonts.google.com/icons)
