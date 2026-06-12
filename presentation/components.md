# Componentes Reutilizables

Componentes UI compartidos utilizados en toda la aplicación.

---

## SearchBar

Barra de búsqueda con icono y texto de ayuda.

**Archivo**: `components/SearchBar.kt`

```kotlin
@Composable
fun SearchBar(
    query: String,
    onQueryChanged: (String) -> Unit,
    placeholder: String,
    modifier: Modifier = Modifier
) {
    OutlinedTextField(
        value = query,
        onValueChange = onQueryChanged,
        placeholder = { Text(placeholder) },
        leadingIcon = {
            Icon(Icons.Default.Search, contentDescription = null)
        },
        trailingIcon = {
            if (query.isNotEmpty()) {
                IconButton(onClick = { onQueryChanged("") }) {
                    Icon(Icons.Default.Clear, contentDescription = "Limpiar")
                }
            }
        },
        singleLine = true,
        modifier = modifier.fillMaxWidth()
    )
}
```

**Características**:
- Icono de búsqueda a la izquierda
- Botón de limpiar (X) que aparece solo cuando hay texto
- Bordes dinámicos con el tema (primary cuando enfocado)
- Una sola línea de texto
- Ocupa todo el ancho disponible

**Uso**:
```kotlin
var searchQuery by remember { mutableStateOf("") }

SearchBar(
    query = searchQuery,
    onQueryChanged = { searchQuery = it },
    placeholder = "Buscar contraseñas..."
)
```

---

## CategoryFilterChips

Chips horizontales para filtrar por categoría.

**Archivo**: `components/CategoryFilterChips.kt`

```kotlin
@Composable
fun CategoryFilterChips(
    categories: List<Category>,
    selectedCategoryId: String?,
    onCategorySelected: (String?) -> Unit,
    onResetAllFilters: () -> Unit,
    modifier: Modifier = Modifier
) {
    Row(modifier = modifier.horizontalScroll(rememberScrollState())) {
        // Chip "Todas"
        FilterChip(
            selected = selectedCategoryId == null,
            onClick = { onResetAllFilters() },
            label = { Text("Todas") }
        )
        
        Spacer(modifier = Modifier.width(8.dp))
        
        // Chips de categorías
        categories.forEach { category ->
            FilterChip(
                selected = selectedCategoryId == category.id,
                onClick = { onCategorySelected(category.id) },
                label = { Text(category.name) },
                colors = FilterChipDefaults.filterChipColors(
                    selectedContainerColor = Color(category.color)
                )
            )
            Spacer(modifier = Modifier.width(8.dp))
        }
    }
}
```

---

## CategoryPicker

Selector de categorías para formularios.

**Archivo**: `components/CategoryPicker.kt`

```kotlin
@Composable
fun CategoryPicker(
    categories: List<Category>,
    selectedCategory: Category?,
    onCategorySelected: (Category) -> Unit,
    modifier: Modifier = Modifier
) {
    var expanded by remember { mutableStateOf(false) }
    
    ExposedDropdownMenuBox(
        expanded = expanded,
        onExpandedChange = { expanded = it }
    ) {
        OutlinedTextField(
            value = selectedCategory?.name ?: "",
            onValueChange = {},
            readOnly = true,
            label = { Text("Categoría") },
            trailingIcon = { ExposedDropdownMenuDefaults.TrailingIcon(expanded = expanded) },
            modifier = modifier.menuAnchor()
        )
        
        ExposedDropdownMenu(
            expanded = expanded,
            onDismissRequest = { expanded = false }
        ) {
            categories.forEach { category ->
                DropdownMenuItem(
                    text = { 
                        Row(verticalAlignment = Alignment.CenterVertically) {
                            Box(
                                modifier = Modifier
                                    .size(16.dp)
                                    .background(Color(category.color), CircleShape)
                            )
                            Spacer(modifier = Modifier.width(8.dp))
                            Text(category.name)
                        }
                    },
                    onClick = {
                        onCategorySelected(category)
                        expanded = false
                    }
                )
            }
        }
    }
}
```

---

## CategoryDropdown

Versión simplificada como dropdown para formularios.

**Archivo**: `components/CategoryPicker.kt`

```kotlin
@Composable
fun CategoryDropdown(
    categories: List<Category>,
    selectedCategory: Category?,
    onCategorySelected: (Category) -> Unit,
    modifier: Modifier = Modifier
) {
    var expanded by remember { mutableStateOf(false) }
    
    ExposedDropdownMenuBox(
        expanded = expanded,
        onExpandedChange = { expanded = !expanded }
    ) {
        OutlinedTextField(
            value = selectedCategory?.name ?: "",
            onValueChange = {},
            readOnly = true,
            label = { Text("Categoría") },
            trailingIcon = { ExposedDropdownMenuDefaults.TrailingIcon(expanded) },
            modifier = Modifier.menuAnchor().fillMaxWidth()
        )
        
        ExposedDropdownMenu(expanded, { expanded = false }) {
            categories.forEach { category ->
                DropdownMenuItem(
                    text = {
                        Row {
                            Box(modifier = Modifier
                                .size(16.dp)
                                .background(Color(category.color), CircleShape))
                            Text(category.name)
                        }
                    },
                    onClick = {
                        onCategorySelected(category)
                        expanded = false
                    }
                )
            }
        }
    }
}
```

**Uso típico**:
```kotlin
CategoryDropdown(
    categories = categories,
    selectedCategory = selectedCategory,
    onCategorySelected = { category ->
        viewModel.onCategorySelected(category)
    }
)
```

---

## ColorPickerDialog

Dialog para seleccionar color de una paleta predefinida.

**Archivo**: `components/ColorPickerDialog.kt`

```kotlin
@Composable
fun ColorPickerDialog(
    currentColor: Int,
    onColorSelected: (Int) -> Unit,
    onDismissRequest: () -> Unit
) {
    var selectedColor by remember { mutableStateOf(currentColor) }
    
    AlertDialog(
        onDismissRequest = onDismissRequest,
        title = { Text("Seleccionar color") },
        text = {
            Column {
                // Vista previa
                Row {
                    Text("Color seleccionado:")
                    Box(modifier = Modifier
                        .size(24.dp)
                        .clip(CircleShape)
                        .background(Color(selectedColor)))
                    Text("#${selectedColor.toString(16).uppercase()}")
                }
                
                // Grid de colores (6 columnas)
                LazyVerticalGrid(
                    columns = GridCells.Fixed(6),
                    modifier = Modifier.height(200.dp)
                ) {
                    items(paletteColors.size) { index ->
                        Box(modifier = Modifier
                            .size(40.dp)
                            .clip(CircleShape)
                            .background(Color(paletteColors[index]))
                            .clickable { selectedColor = paletteColors[index] }
                            .then(if (selectedColor == paletteColors[index])
                                Modifier.padding(3.dp)
                                    .background(MaterialTheme.colorScheme.onSurface, CircleShape)
                                else Modifier))
                    }
                }
            }
        },
        confirmButton = {
            TextButton(onClick = { 
                onColorSelected(selectedColor)
                onDismissRequest()
            }) { Text("Aceptar") }
        },
        dismissButton = {
            TextButton(onClick = onDismissRequest) { Text("Cancelar") }
        }
    )
}
```

**Paleta de Colores** (28 colores):

| Grupo | Colores (HEX) |
|-------|---------------|
| Rojos | #E74C3C, #C0392B, #EA4335 |
| Naranjas | #F39C12, #E67E22, #D35400 |
| Amarillos | #F1C40F, #FECA57, #FF9F43 |
| Verdes | #27AE60, #2ECC71, #1ABC9C |
| Azules | #3498DB, #2980B9, #4267B2, #5DADE2 |
| Morados | #8E44AD, #9B59B6, #A569BD |
| Rosas | #E91E63, #FF4081, #FF69B4 |
| Grises | #95A5A6, #7F8C8D, #BDC3C7, #34495E |
| Extra | #000000, #FFFFFF |

---

## PasswordGeneratorWidget

Widget para generar contraseñas dentro de formularios.

**Archivo**: `components/PasswordGeneratorWidget.kt`

```kotlin
@Composable
fun PasswordGeneratorWidget(
    onGenerate: (PasswordOptions) -> String,
    onPasswordGenerated: (String) -> Unit,
    modifier: Modifier = Modifier
) {
    var showOptions by remember { mutableStateOf(false) }
    var options by remember { mutableStateOf(PasswordOptions()) }
    var generatedPassword by remember { mutableStateOf("") }
    
    Column(modifier = modifier) {
        Row(
            horizontalArrangement = Arrangement.spacedBy(8.dp),
            modifier = Modifier.fillMaxWidth()
        ) {
            Button(
                onClick = { showOptions = true },
                modifier = Modifier.weight(1f)
            ) {
                Icon(Icons.Default.Settings, contentDescription = null)
                Spacer(modifier = Modifier.width(8.dp))
                Text("Opciones")
            }
            
            Button(
                onClick = {
                    generatedPassword = onGenerate(options)
                    onPasswordGenerated(generatedPassword)
                },
                modifier = Modifier.weight(1f)
            ) {
                Icon(Icons.Default.Refresh, contentDescription = null)
                Spacer(modifier = Modifier.width(8.dp))
                Text("Generar")
            }
        }
        
        if (generatedPassword.isNotEmpty()) {
            Spacer(modifier = Modifier.height(8.dp))
            OutlinedTextField(
                value = generatedPassword,
                onValueChange = {},
                readOnly = true,
                label = { Text("Contraseña generada") },
                trailingIcon = {
                    IconButton(onClick = { /* Copiar al portapapeles */ }) {
                        Icon(Icons.Default.ContentCopy, contentDescription = "Copiar")
                    }
                },
                modifier = Modifier.fillMaxWidth()
            )
        }
        
        if (showOptions) {
            PasswordOptionsDialog(
                options = options,
                onOptionsChange = { options = it },
                onDismiss = { showOptions = false }
            )
        }
    }
}
```
