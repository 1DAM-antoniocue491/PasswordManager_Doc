# ColorPickerDialog

## Visión General

Dialog para seleccionar un color de una paleta predefinida.

**Archivo**: `presentation/ui/components/ColorPickerDialog.kt`

## Uso

```kotlin
@Composable
fun CategoryFormDialog(...) {
    var selectedColor by remember { mutableStateOf(0xFF3498DB.toInt()) }
    var showColorPicker by remember { mutableStateOf(false) }
    
    if (showColorPicker) {
        ColorPickerDialog(
            currentColor = selectedColor,
            onColorSelected = { selectedColor = it },
            onDismissRequest = { showColorPicker = false }
        )
    }
    
    Button(onClick = { showColorPicker = true }) {
        Box(
            modifier = Modifier
                .size(24.dp)
                .background(Color(selectedColor), CircleShape)
        )
        Spacer(modifier = Modifier.width(8.dp))
        Text("Seleccionar color")
    }
}
```

## API

```kotlin
@Composable
fun ColorPickerDialog(
    currentColor: Int,                    // Color actual (ARGB Int)
    onColorSelected: (Int) -> Unit,       // Callback de selección
    onDismissRequest: () -> Unit          // Callback de dismiss
)
```

## Paleta de Colores

```kotlin
private val paletteColors = listOf(
    // Rojos
    0xFFE74C3C.toInt(),
    0xFFC0392B.toInt(),
    0xFFEA4335.toInt(),
    // Naranjas
    0xFFF39C12.toInt(),
    0xFFE67E22.toInt(),
    0xFFD35400.toInt(),
    // Amarillos
    0xFFF1C40F.toInt(),
    0xFFFECA57.toInt(),
    0xFFFF9F43.toInt(),
    // Verdes
    0xFF27AE60.toInt(),
    0xFF2ECC71.toInt(),
    0xFF1ABC9C.toInt(),
    // Azules
    0xFF3498DB.toInt(),
    0xFF2980B9.toInt(),
    0xFF4267B2.toInt(),
    0xFF5DADE2.toInt(),
    // Morados
    0xFF8E44AD.toInt(),
    0xFF9B59B6.toInt(),
    0xFFA569BD.toInt(),
    // Rosas
    0xFFE91E63.toInt(),
    0xFFFF4081.toInt(),
    0xFFFF69B4.toInt(),
    // Grises
    0xFF95A5A6.toInt(),
    0xFF7F8C8D.toInt(),
    0xFFBDC3C7.toInt(),
    0xFF34495E.toInt(),
    // Extra
    0xFF000000.toInt(),
    0xFFFFFFFF.toInt()
)
```

## Implementación

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
                // Vista previa del color seleccionado
                Row(
                    verticalAlignment = Alignment.CenterVertically,
                    modifier = Modifier.padding(bottom = 16.dp)
                ) {
                    Text(
                        text = "Color seleccionado:",
                        style = MaterialTheme.typography.labelMedium
                    )
                    Spacer(modifier = Modifier.width(8.dp))
                    Box(
                        modifier = Modifier
                            .size(24.dp)
                            .clip(CircleShape)
                            .background(Color(selectedColor))
                    )
                    Spacer(modifier = Modifier.width(8.dp))
                    Text(
                        text = "#${selectedColor.toString(16).uppercase().removePrefix("FF")}",
                        style = MaterialTheme.typography.labelSmall
                    )
                }
                
                // Grid de colores (6 columnas)
                LazyVerticalGrid(
                    columns = GridCells.Fixed(6),
                    horizontalArrangement = Arrangement.spacedBy(8.dp),
                    verticalArrangement = Arrangement.spacedBy(8.dp),
                    modifier = Modifier.height(200.dp)
                ) {
                    items(paletteColors.size) { index ->
                        val color = paletteColors[index]
                        Box(
                            modifier = Modifier
                                .size(40.dp)
                                .clip(CircleShape)
                                .background(Color(color))
                                .clickable { selectedColor = color }
                                .then(
                                    if (selectedColor == color) {
                                        Modifier
                                            .padding(3.dp)
                                            .background(
                                                color = MaterialTheme.colorScheme.onSurface,
                                                shape = CircleShape
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
                    onColorSelected(selectedColor)
                    onDismissRequest()
                }
            ) {
                Text("Aceptar")
            }
        },
        dismissButton = {
            TextButton(onClick = onDismissRequest) {
                Text("Cancelar")
            }
        }
    )
}
```

## Referencias

- [CategoryPicker](./category-picker.md)
- [CategoryManagementScreen](../screens/category-management.md)
