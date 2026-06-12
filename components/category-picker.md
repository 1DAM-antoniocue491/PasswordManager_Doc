# CategoryPicker

## Visión General

Componente para seleccionar una categoría desde un dialog con grid de opciones.

**Archivo**: `presentation/ui/components/CategoryPicker.kt`

## Uso Básico

```kotlin
@Composable
fun PasswordFormScreen(...) {
    var selectedCategory by remember { mutableStateOf<Category?>(null) }
    var showCategoryPicker by remember { mutableStateOf(false) }
    
    // Mostrar picker cuando sea necesario
    if (showCategoryPicker) {
        CategoryPicker(
            categories = categories,
            selectedCategory = selectedCategory,
            onCategorySelected = { 
                selectedCategory = it
                showCategoryPicker = false
            },
            onDismissRequest = { showCategoryPicker = false }
        )
    }
    
    // Trigger para abrir el picker
    OutlinedButton(onClick = { showCategoryPicker = true }) {
        Text(selectedCategory?.name ?: "Seleccionar categoría")
    }
}
```

## API del Componente

```kotlin
@Composable
fun CategoryPicker(
    categories: List<Category>,                   // Categorías disponibles
    selectedCategory: Category?,                  // Categoría seleccionada
    onCategorySelected: (Category) -> Unit,       // Callback de selección
    onDismissRequest: () -> Unit,                 // Callback de dismiss
    onCreateCategory: ((String, Int, String) -> Unit)? = null,  // Opcional: crear nueva
    modifier: Modifier = Modifier
)
```

## Estructura Visual

```
┌─────────────────────────────────────────┐
│  Seleccionar categoría                  │
├─────────────────────────────────────────┤
│                                         │
│  ┌──────┐  ┌──────┐  ┌──────┐          │
│  │  📁  │  │  👥  │  │  💰  │          │
│  │General│ │Social│ │Finan │          │
│  └──────┘  └──────┘  └──────┘          │
│                                         │
│  ┌──────┐  ┌──────┐  ┌──────┐          │
│  │  🛒  │  │  🎬  │  │  💼  │          │
│  │Compra│ │Entre │ │Trab  │          │
│  └──────┘  └──────┘  └──────┘          │
│                                         │
│  ┌──────┐  ┌──────┐  ┌──────┐          │
│  │  🎓  │  │  ❤️  │  │  ✈️  │          │
│  │Educ  │ │Salud │ │Viaje │          │
│  └──────┘  └──────┘  └──────┘          │
│                                         │
│  ┌──────┐                              │
│  │  +   │  (Nueva categoría)           │
│  └──────┘                              │
│                                         │
│         [Cancelar]                      │
└─────────────────────────────────────────┘
```

## Implementación

```kotlin
@Composable
fun CategoryPicker(
    categories: List<Category>,
    selectedCategory: Category?,
    onCategorySelected: (Category) -> Unit,
    onDismissRequest: () -> Unit,
    onCreateCategory: ((String, Int, String) -> Unit)? = null,
    modifier: Modifier = Modifier
) {
    var showNewCategoryDialog by remember { mutableStateOf(false) }
    
    if (showNewCategoryDialog) {
        NewCategoryDialog(
            onConfirm = { name, color, icon ->
                onCreateCategory?.invoke(name, color, icon)
                showNewCategoryDialog = false
            },
            onDismiss = { showNewCategoryDialog = false }
        )
    }
    
    AlertDialog(
        onDismissRequest = onDismissRequest,
        title = { Text("Seleccionar categoría") },
        text = {
            LazyVerticalGrid(
                columns = GridCells.Fixed(3),
                horizontalArrangement = Arrangement.spacedBy(8.dp),
                verticalArrangement = Arrangement.spacedBy(8.dp)
            ) {
                items(categories.size) { index ->
                    CategoryItem(
                        category = categories[index],
                        isSelected = selectedCategory?.id == categories[index].id,
                        onClick = { onCategorySelected(categories[index]) }
                    )
                }
                
                if (onCreateCategory != null) {
                    item {
                        AddCategoryItem(onClick = { showNewCategoryDialog = true })
                    }
                }
            }
        },
        confirmButton = {
            TextButton(onClick = onDismissRequest) {
                Text("Cancelar")
            }
        }
    )
}

@Composable
private fun CategoryItem(
    category: Category,
    isSelected: Boolean,
    onClick: () -> Unit
) {
    Column(
        horizontalAlignment = Alignment.CenterHorizontally,
        modifier = Modifier
            .clickable(onClick = onClick)
            .padding(8.dp)
    ) {
        Box(
            contentAlignment = Alignment.Center,
            modifier = Modifier
                .size(48.dp)
                .background(
                    color = Color(category.color),
                    shape = CircleShape
                )
                .then(
                    if (isSelected) {
                        Modifier
                            .padding(2.dp)
                            .background(
                                color = MaterialTheme.colorScheme.primary,
                                shape = CircleShape
                            )
                    } else {
                        Modifier
                    }
                )
        ) {
            Icon(
                imageVector = Icons.Default.Star,
                contentDescription = category.name,
                tint = Color.White,
                modifier = Modifier.size(24.dp)
            )
        }
        
        Spacer(modifier = Modifier.height(4.dp))
        
        Text(
            text = category.name,
            style = MaterialTheme.typography.labelSmall,
            maxLines = 1
        )
    }
}
```

## CategoryDropdown (Variante)

Versión simplificada como dropdown para formularios:

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
            trailingIcon = { 
                ExposedDropdownMenuDefaults.TrailingIcon(expanded = expanded)
            },
            modifier = Modifier.menuAnchor().fillMaxWidth()
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
                                    .background(
                                        color = Color(category.color),
                                        shape = CircleShape
                                    )
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

## NewCategoryDialog

Dialog para crear una nueva categoría:

```kotlin
@Composable
private fun NewCategoryDialog(
    onConfirm: (String, Int, String) -> Unit,
    onDismiss: () -> Unit
) {
    var categoryName by remember { mutableStateOf("") }
    var selectedColor by remember { mutableStateOf(Color(0xFF3498DB)) }
    var selectedIcon by remember { mutableStateOf("FaFolder") }
    
    AlertDialog(
        onDismissRequest = onDismiss,
        title = { Text("Nueva categoría") },
        text = {
            Column {
                // Nombre
                OutlinedTextField(
                    value = categoryName,
                    onValueChange = { categoryName = it },
                    label = { Text("Nombre") },
                    singleLine = true
                )
                
                Spacer(modifier = Modifier.height(16.dp))
                
                // Selector de color
                Text("Color:", style = MaterialTheme.typography.labelMedium)
                Spacer(modifier = Modifier.height(8.dp))
                Row {
                    listOf(
                        Color(0xFF3498DB),
                        Color(0xFF4267B2),
                        Color(0xFFEA4335),
                        Color(0xFF27AE60),
                        Color(0xFFF39C12),
                        Color(0xFF8E44AD)
                    ).forEach { color ->
                        Box(
                            modifier = Modifier
                                .size(32.dp)
                                .background(color, CircleShape)
                                .clickable { selectedColor = color }
                                .then(
                                    if (selectedColor == color) {
                                        Modifier
                                            .padding(2.dp)
                                            .background(
                                                MaterialTheme.colorScheme.primary,
                                                CircleShape
                                            )
                                    } else {
                                        Modifier
                                    }
                                )
                        )
                    }
                }
            }
        },
        confirmButton = {
            TextButton(
                onClick = {
                    if (categoryName.isNotBlank()) {
                        onConfirm(categoryName.trim(), selectedColor.toArgb(), selectedIcon)
                    }
                },
                enabled = categoryName.isNotBlank()
            ) {
                Text("Crear")
            }
        },
        dismissButton = {
            TextButton(onClick = onDismiss) {
                Text("Cancelar")
            }
        }
    )
}
```

## Referencias

- [CategoryManagementScreen](../screens/category-management.md)
- [CategoryPicker](./category-picker.md)
- [ColorPickerDialog](./color-picker.md)
