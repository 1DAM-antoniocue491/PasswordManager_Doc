# Componentes Reutilizables

## Visión General

La aplicación utiliza componentes reutilizables de Jetpack Compose para mantener consistencia visual y reducir código duplicado.

## Ubicación

```
presentation/ui/components/
├── SearchBar.kt
├── CategoryFilterChips.kt
├── CategoryPicker.kt
├── PasswordGeneratorWidget.kt
└── ColorPickerDialog.kt
```

## Componentes Disponibles

| Componente | Uso | Estado |
|------------|-----|--------|
| `SearchBar` | Búsqueda en listas | ✅ Estable |
| `CategoryFilterChips` | Filtro de categorías | ✅ Estable |
| `CategoryPicker` | Selector de categorías | ✅ Estable |
| `PasswordGeneratorWidget` | Generador en formularios | ✅ Estable |
| `ColorPickerDialog` | Selector de color | ✅ Estable |

---

## SearchBar

### Descripción

Barra de búsqueda con icono, texto de ayuda y botón de limpiar automático.

### Archivo

`SearchBar.kt`

### API

```kotlin
@Composable
fun SearchBar(
    query: String,
    onQueryChanged: (String) -> Unit,
    placeholder: String = "Buscar...",
    modifier: Modifier = Modifier,
    singleLine: Boolean = true
)
```

### Parámetros

| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `query` | `String` | - | Texto actual de búsqueda |
| `onQueryChanged` | `(String) -> Unit` | - | Callback cuando cambia el texto |
| `placeholder` | `String` | `"Buscar..."` | Texto de ayuda |
| `modifier` | `Modifier` | `Modifier` | Modificador opcional |
| `singleLine` | `Boolean` | `true` | Una sola línea o multilínea |

### Uso

```kotlin
var searchQuery by remember { mutableStateOf("") }

SearchBar(
    query = searchQuery,
    onQueryChanged = { searchQuery = it },
    placeholder = "Buscar por título, usuario o URL...",
    modifier = Modifier.padding(horizontal = 16.dp, vertical = 8.dp)
)
```

### Implementación

```kotlin
@Composable
fun SearchBar(
    query: String,
    onQueryChanged: (String) -> Unit,
    placeholder: String,
    modifier: Modifier = Modifier,
    singleLine: Boolean = true
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
                        imageVector = Icons.Default.Clear,
                        contentDescription = "Limpiar búsqueda",
                        tint = MaterialTheme.colorScheme.onSurfaceVariant
                    )
                }
            }
        },
        singleLine = singleLine,
        modifier = modifier.fillMaxWidth(),
        colors = OutlinedTextFieldDefaults.colors(
            focusedBorderColor = MaterialTheme.colorScheme.primary,
            unfocusedBorderColor = MaterialTheme.colorScheme.outline
        )
    )
}
```

### Visualización

```
┌────────────────────────────────────────────────────┐
│ 🔍 Buscar por título, usuario o URL...         ✕   │
└────────────────────────────────────────────────────┘
```

**Estados**:
- **Vacío**: Muestra placeholder, sin botón de limpiar
- **Con texto**: Muestra texto, botón de limpiar visible
- **Foco**: Borde color primario

---

## CategoryFilterChips

### Descripción

Fila horizontal de chips para filtrar por categoría con scroll.

### Archivo

`CategoryFilterChips.kt`

### API

```kotlin
@Composable
fun CategoryFilterChips(
    categories: List<Category>,
    selectedCategoryId: String?,
    onCategorySelected: (String?) -> Unit,
    onResetAllFilters: () -> Unit,
    modifier: Modifier = Modifier
)
```

### Parámetros

| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `categories` | `List<Category>` | - | Lista de categorías |
| `selectedCategoryId` | `String?` | - | ID seleccionada (null = todas) |
| `onCategorySelected` | `(String?) -> Unit` | - | Callback de selección |
| `onResetAllFilters` | `() -> Unit` | - | Resetear todos los filtros |
| `modifier` | `Modifier` | `Modifier` | Modificador opcional |

### Uso

```kotlin
var selectedCategory by remember { mutableStateOf<String?>(null) }
val categories by viewModel.categories.collectAsState()

CategoryFilterChips(
    categories = categories,
    selectedCategoryId = selectedCategory,
    onCategorySelected = { selectedCategory = it },
    onResetAllFilters = { selectedCategory = null },
    modifier = Modifier.padding(horizontal = 16.dp, vertical = 8.dp)
)
```

### Implementación

```kotlin
@Composable
fun CategoryFilterChips(
    categories: List<Category>,
    selectedCategoryId: String?,
    onCategorySelected: (String?) -> Unit,
    onResetAllFilters: () -> Unit,
    modifier: Modifier = Modifier
) {
    Row(
        modifier = modifier
            .fillMaxWidth()
            .horizontalScroll(rememberScrollState()),
        horizontalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        // Chip "Todas"
        FilterChip(
            selected = selectedCategoryId == null,
            onClick = onResetAllFilters,
            label = { Text("Todas") },
            leadingIcon = if (selectedCategoryId == null) {
                {
                    Icon(
                        imageVector = Icons.Default.Check,
                        contentDescription = null,
                        modifier = Modifier.size(18.dp)
                    )
                }
            } else null
        )
        
        // Chips de categorías
        categories.forEach { category ->
            FilterChip(
                selected = selectedCategoryId == category.id,
                onClick = { onCategorySelected(category.id) },
                label = { Text(category.name) },
                colors = FilterChipDefaults.filterChipColors(
                    selectedContainerColor = Color(category.color)
                ),
                leadingIcon = if (selectedCategoryId == category.id) {
                    {
                        Icon(
                            imageVector = Icons.Default.Check,
                            contentDescription = null,
                            modifier = Modifier.size(18.dp)
                        )
                    }
                } else null
            )
        }
    }
}
```

### Visualización

```
┌──────────────────────────────────────────────────────────────┐
│ [✓ Todas] [General] [🔵 Redes] [🟢 Finanzas] [🟠 Compras] →  │
└──────────────────────────────────────────────────────────────┘
```

**Comportamiento**:
- Scroll horizontal si hay muchas categorías
- Chip seleccionado muestra color de categoría
- Icono de check en seleccionado

---

## CategoryPicker

### Descripción

Selector desplegable de categorías para formularios.

### Archivo

`CategoryPicker.kt`

### API

```kotlin
@Composable
fun CategoryPicker(
    categories: List<Category>,
    selectedCategory: Category?,
    onCategorySelected: (Category) -> Unit,
    modifier: Modifier = Modifier,
    label: String = "Categoría"
)
```

### Parámetros

| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `categories` | `List<Category>` | - | Lista de categorías |
| `selectedCategory` | `Category?` | - | Categoría seleccionada |
| `onCategorySelected` | `(Category) -> Unit` | - | Callback de selección |
| `modifier` | `Modifier` | `Modifier` | Modificador opcional |
| `label` | `String` | `"Categoría"` | Etiqueta del campo |

### Uso

```kotlin
var selectedCategory by remember { mutableStateOf<Category?>(null) }
val categories by viewModel.categories.collectAsState()

CategoryPicker(
    categories = categories,
    selectedCategory = selectedCategory,
    onCategorySelected = { selectedCategory = it },
    modifier = Modifier.fillMaxWidth(),
    label = "Categoría"
)
```

### Implementación

```kotlin
@Composable
fun CategoryPicker(
    categories: List<Category>,
    selectedCategory: Category?,
    onCategorySelected: (Category) -> Unit,
    modifier: Modifier = Modifier,
    label: String = "Categoría"
) {
    var expanded by remember { mutableStateOf(false) }
    
    Box(modifier = modifier) {
        ExposedDropdownMenuBox(
            expanded = expanded,
            onExpandedChange = { expanded = it }
        ) {
            OutlinedTextField(
                value = selectedCategory?.name ?: "",
                onValueChange = {},
                readOnly = true,
                label = { Text(label) },
                trailingIcon = {
                    ExposedDropdownMenuDefaults.TrailingIcon(expanded = expanded)
                },
                colors = OutlinedTextFieldDefaults.colors(
                    focusedBorderColor = if (expanded) {
                        MaterialTheme.colorScheme.primary
                    } else {
                        MaterialTheme.colorScheme.outline
                    }
                ),
                modifier = Modifier
                    .fillMaxWidth()
                    .menuAnchor()
            )
            
            ExposedDropdownMenu(
                expanded = expanded,
                onDismissRequest = { expanded = false }
            ) {
                categories.forEach { category ->
                    DropdownMenuItem(
                        text = {
                            Row(verticalAlignment = Alignment.CenterVertically) {
                                // Indicador de color
                                Box(
                                    modifier = Modifier
                                        .size(16.dp)
                                        .background(
                                            Color(category.color),
                                            CircleShape
                                        )
                                )
                                Spacer(modifier = Modifier.width(8.dp))
                                Text(category.name)
                            }
                        },
                        onClick = {
                            onCategorySelected(category)
                            expanded = false
                        },
                        leadingIcon = if (category == selectedCategory) {
                            {
                                Icon(
                                    imageVector = Icons.Default.Check,
                                    contentDescription = null
                                )
                            }
                        } else null
                    )
                }
            }
        }
    }
}
```

### Visualización

**Cerrado**:
```
┌─────────────────────────────────────┐
│ Categoría                           │
│ General                         ▼   │
└─────────────────────────────────────┘
```

**Abierto**:
```
┌─────────────────────────────────────┐
│ Categoría                           │
│ General                         ▲   │
├─────────────────────────────────────┤
│ ● General                      ✓    │
│ 🔵 Redes Sociales                   │
│ 🟢 Finanzas                         │
│ 🟠 Compras                          │
│ 🔴 Entretenimiento                  │
└─────────────────────────────────────┘
```

---

## PasswordGeneratorWidget

### Descripción

Widget para generar contraseñas seguras dentro de formularios.

### Archivo

`PasswordGeneratorWidget.kt`

### API

```kotlin
@Composable
fun PasswordGeneratorWidget(
    onGenerate: (PasswordOptions) -> String,
    onPasswordGenerated: (String) -> Unit,
    modifier: Modifier = Modifier,
    initialOptions: PasswordOptions = PasswordOptions()
)
```

### Parámetros

| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `onGenerate` | `(PasswordOptions) -> String` | - | Función para generar |
| `onPasswordGenerated` | `(String) -> Unit` | - | Callback con password generada |
| `modifier` | `Modifier` | `Modifier` | Modificador opcional |
| `initialOptions` | `PasswordOptions` | `PasswordOptions()` | Opciones iniciales |

### Uso

```kotlin
PasswordGeneratorWidget(
    onGenerate = { options ->
        PasswordGenerator.generate(options)
    },
    onPasswordGenerated = { password ->
        viewModel.onFieldChange(FormField.PASSWORD, password)
    },
    modifier = Modifier.fillMaxWidth()
)
```

### Visualización

```
┌─────────────────────────────────────────────────────┐
│ [⚙️ Opciones]           [🔄 Generar]                │
├─────────────────────────────────────────────────────┤
│ Contraseña generada                                 │
│ K#9mX$2pL@5nQr8Y                    [📋 Copiar]     │
└─────────────────────────────────────────────────────┘
```

### Diálogo de Opciones

```
┌───────────────────────────────────┐
│  Opciones de generación           │
├───────────────────────────────────┤
│  Longitud: 16                     │
│  ├────────●────────┤ 8-128        │
│                                   │
│  ☑ Mayúsculas (A-Z)               │
│  ☑ Minúsculas (a-z)               │
│  ☑ Números (0-9)                  │
│  ☑ Símbolos (!@#$...)             │
├───────────────────────────────────┤
│            [Aceptar]              │
└───────────────────────────────────┘
```

---

## ColorPickerDialog

### Descripción

Diálogo para seleccionar color de categoría de una paleta predefinida.

### Archivo

`ColorPickerDialog.kt`

### API

```kotlin
@Composable
fun ColorPickerDialog(
    selectedColor: Int,
    onColorSelected: (Int) -> Unit,
    onDismiss: () -> Unit,
    colors: List<Int> = DefaultColorPalette
)
```

### Parámetros

| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `selectedColor` | `Int` | - | Color actual seleccionado |
| `onColorSelected` | `(Int) -> Unit` | - | Callback con color seleccionado |
| `onDismiss` | `() -> Unit` | - | Cerrar diálogo |
| `colors` | `List<Int>` | `DefaultColorPalette` | Paleta de colores |

### Paleta por Defecto

```kotlin
val DefaultColorPalette = listOf(
    0xFFFF5722, // Rojo anaranjado
    0xFFE91E63, // Rosa
    0xFF9C27B0, // Púrpura
    0xFF673AB7, // Violeta oscuro
    0xFF3F51B5, // Índigo
    0xFF2196F3, // Azul
    0xFF03A9F4, // Azul claro
    0xFF00BCD4, // Cyan
    0xFF009688, // Teal
    0xFF4CAF50, // Verde
    0xFF8BC34A, // Verde claro
    0xFFCDDC39, // Lima
    0xFFFFEB3B, // Amarillo
    0xFFFFC107, // Ámbar
    0xFFFF9800, // Naranja
    0xFF795548  // Marrón
)
```

### Uso

```kotlin
var showColorPicker by remember { mutableStateOf(false) }
var selectedColor by remember { mutableStateOf(0xFF9E9E9E.toInt()) }

if (showColorPicker) {
    ColorPickerDialog(
        selectedColor = selectedColor,
        onColorSelected = { selectedColor = it },
        onDismiss = { showColorPicker = false }
    )
}

Button(onClick = { showColorPicker = true }) {
    Text("Seleccionar color")
}
```

### Implementación

```kotlin
@Composable
fun ColorPickerDialog(
    selectedColor: Int,
    onColorSelected: (Int) -> Unit,
    onDismiss: () -> Unit,
    colors: List<Int> = DefaultColorPalette
) {
    AlertDialog(
        onDismissRequest = onDismiss,
        title = { Text("Seleccionar color") },
        text = {
            LazyVerticalGrid(
                columns = GridCells.Fixed(4),
                horizontalArrangement = Arrangement.spacedBy(8.dp),
                verticalArrangement = Arrangement.spacedBy(8.dp)
            ) {
                items(colors) { color ->
                    IconButton(
                        onClick = {
                            onColorSelected(color)
                            onDismiss()
                        }
                    ) {
                        Box(
                            modifier = Modifier
                                .size(48.dp)
                                .background(Color(color), CircleShape),
                            contentAlignment = Alignment.Center
                        ) {
                            if (color == selectedColor) {
                                Icon(
                                    imageVector = Icons.Default.Check,
                                    contentDescription = null,
                                    tint = Color.White,
                                    modifier = Modifier.size(24.dp)
                                )
                            }
                        }
                    }
                }
            }
        },
        confirmButton = {
            TextButton(onClick = onDismiss) {
                Text("Cancelar")
            }
        }
    )
}
```

### Visualización

```
┌───────────────────────────────────────┐
│  Seleccionar color                    │
├───────────────────────────────────────┤
│  🔴  🩷  🟣  🟙                        │
│  🟠  🔵  🔷  🔶                        │
│  🟢  🟩  🟨  🟧                        │
│  🟤  ⚫  ⚪  🟫                        │
├───────────────────────────────────────┤
│              [Cancelar]               │
└───────────────────────────────────────┘
```

---

## Guías de Uso

### Cuándo Usar Cada Componente

| Escenario | Componente Recomendado |
|-----------|----------------------|
| Búsqueda en lista | `SearchBar` |
| Filtrar por categoría | `CategoryFilterChips` |
| Seleccionar categoría en formulario | `CategoryPicker` |
| Generar contraseña en formulario | `PasswordGeneratorWidget` |
| Crear/editar categoría | `ColorPickerDialog` |

### Mejores Prácticas

1. **SearchBar**: Siempre con placeholder descriptivo
2. **CategoryFilterChips**: Incluir siempre chip "Todas"
3. **CategoryPicker**: Mostrar color de categoría en dropdown
4. **PasswordGeneratorWidget**: Validar al menos un tipo de carácter
5. **ColorPickerDialog**: Usar grid 4x4 para mejor visualización

### Accesibilidad

Todos los componentes incluyen:
- `contentDescription` en iconos
- `label` en campos de texto
- `stateDescription` para estados
- Tamaño de toque mínimo 48dp

---

**Documentación Relacionada:**
- [Screens Overview](../screens/overview.md)
- [Theme](../presentation/theme.md)
- [Material 3 Components](https://m3.material.io/components)
