# PasswordFormScreen

## Descripción

Pantalla para crear nuevas contraseñas o editar existentes. Contiene un formulario completo con validación y un generador de contraseñas integrado.

**Archivo**: `PasswordFormScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  ← Nueva Contraseña              ✓     │
├─────────────────────────────────────────┤
│                                         │
│  Título *                               │
│  ┌─────────────────────────────────┐   │
│  │ Mi cuenta de Google             │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Username/Email *                       │
│  ┌─────────────────────────────────┐   │
│  │ usuario@gmail.com               │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Contraseña *                           │
│  ┌─────────────────────────────────┐   │
│  │ ••••••••••••••••        👁️ 👤   │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │ [⚙️ Opciones] [🔄 Generar]     │   │
│  └─────────────────────────────────┘   │
│                                         │
│  URL                                    │
│  ┌─────────────────────────────────┐   │
│  │ https://google.com              │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Notas                                  │
│  ┌─────────────────────────────────┐   │
│  │ Cuenta personal creada en...    │   │
│  │                                 │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Categoría                              │
│  ┌─────────────────────────────────┐   │
│  │ General                    ▼    │   │
│  └─────────────────────────────────┘   │
│                                         │
│  □ Marcar como favorito                 │
│                                         │
└─────────────────────────────────────────┘
```

## Campos del Formulario

| Campo | Tipo | Requerido | Validación |
|-------|------|-----------|------------|
| Título | TextField | Sí | No vacío |
| Username/Email | TextField | Sí | No vacío |
| Contraseña | TextField | Sí | No vacío |
| URL | TextField | No | Formato URL válido |
| Notas | TextField | No | - |
| Categoría | Selector | Sí | Debe haber una seleccionada |
| Favorito | Checkbox | No | - |

## Estado (State)

```kotlin
data class PasswordFormState(
    val title: String = "",
    val username: String = "",
    val password: String = "",
    val url: String = "",
    val notes: String = "",
    val selectedCategory: Category? = null,
    val isFavorite: Boolean = false,
    val error: String? = null,
    val isSaving: Boolean = false,
    val showPassword: Boolean = false
)
```

## Validaciones

### En Tiempo Real

```kotlin
val isFormValid = state.title.isNotBlank() &&
                  state.username.isNotBlank() &&
                  state.password.isNotBlank() &&
                  state.selectedCategory != null &&
                  (state.url.isEmpty() || isValidUrl(state.url))
```

### Mensajes de Error

| Campo | Error | Mensaje |
|-------|-------|---------|
| Título | Vacío | "El título es requerido" |
| Username | Vacío | "El username es requerido" |
| Contraseña | Vacío | "La contraseña es requerida" |
| URL | Inválido | "La URL no es válida" |
| Categoría | Null | "Debe seleccionar una categoría" |

## Componentes

### PasswordField con Toggle

```kotlin
OutlinedTextField(
    value = state.password,
    onValueChange = { onFieldChange(FormField.PASSWORD, it) },
    label = { Text("Contraseña") },
    visualTransformation = if (state.showPassword) {
        VisualTransformation.None
    } else {
        PasswordVisualTransformation()
    },
    trailingIcon = {
        IconButton(onClick = { onTogglePasswordVisibility() }) {
            Icon(
                imageVector = if (state.showPassword) {
                    Icons.Default.Visibility
                } else {
                    Icons.Default.VisibilityOff
                },
                contentDescription = if (state.showPassword) {
                    "Ocultar contraseña"
                } else {
                    "Mostrar contraseña"
                }
            )
        }
    }
)
```

### Category Selector

```kotlin
Box(modifier = Modifier.fillMaxWidth().clickable(onClick = onOpenPicker)) {
    OutlinedTextField(
        value = selectedCategoryName,
        onValueChange = {},
        readOnly = true,
        label = { Text("Categoría") },
        trailingIcon = {
            Icon(Icons.Default.ArrowDropDown, contentDescription = null)
        }
    )
}

// Diálogo de selección
CategoryPickerDialog(
    categories = categories,
    selectedCategory = state.selectedCategory,
    onCategorySelected = { onCategorySelected(it) },
    onDismiss = { showDialog = false }
)
```

### Generador Integrado

```kotlin
Row(
    horizontalArrangement = Arrangement.spacedBy(8.dp),
    modifier = Modifier.fillMaxWidth()
) {
    Button(
        onClick = { showGeneratorOptions = true },
        modifier = Modifier.weight(1f).heightIn(min = 48.dp),
        contentPadding = PaddingValues(horizontal = 12.dp, vertical = 8.dp)
    ) {
        Icon(Icons.Default.Settings, contentDescription = null, modifier = Modifier.size(18.dp))
        Spacer(modifier = Modifier.width(8.dp))
        Text("Opciones")
    }
    
    Button(
        onClick = {
            val generated = generatePassword(options)
            onFieldChange(FormField.PASSWORD, generated)
        },
        modifier = Modifier.weight(1f).heightIn(min = 48.dp),
        contentPadding = PaddingValues(horizontal = 12.dp, vertical = 8.dp)
    ) {
        Icon(Icons.Default.Refresh, contentDescription = null, modifier = Modifier.size(18.dp))
        Spacer(modifier = Modifier.width(8.dp))
        Text("Generar")
    }
}
```

## Eventos de UI

| Evento | Parámetros | Descripción |
|--------|------------|-------------|
| `onFieldChange` | `field: FormField, value: String` | Cambiar valor de campo |
| `onTogglePasswordVisibility` | - | Mostrar/ocultar password |
| `onCategorySelected` | `category: Category` | Seleccionar categoría |
| `onToggleFavorite` | - | Marcar/desmarcar favorito |
| `onSave` | - | Guardar contraseña |
| `onGeneratePassword` | `options: PasswordOptions` | Generar contraseña |

## Flujo de Guardado

```
Usuario pulsa "Guardar" (✓)
         ↓
Validar todos los campos
         ↓
¿Válido? ──No──> Mostrar error
         │
         Sí
         ↓
Crear PasswordEntry
         ↓
ViewModel.createPasswordEntry(entry)
         ↓
UseCase → Repository → DAO
         ↓
¿Éxito? ──No──> Mostrar error
         │
         Sí
         ↓
Navegar atrás (popBackStack)
```

## Código de Guardado

```kotlin
fun onSave() {
    viewModelScope.launch {
        // Validar campos
        when {
            state.title.isBlank() -> {
                _state.value = _state.value.copy(
                    error = "El título es requerido"
                )
                return@launch
            }
            state.username.isBlank() -> {
                _state.value = _state.value.copy(
                    error = "El username es requerido"
                )
                return@launch
            }
            state.password.isBlank() -> {
                _state.value = _state.value.copy(
                    error = "La contraseña es requerida"
                )
                return@launch
            }
            state.selectedCategory == null -> {
                _state.value = _state.value.copy(
                    error = "Debe seleccionar una categoría"
                )
                return@launch
            }
        }
        
        // Crear entrada
        val entry = PasswordEntry(
            id = UUID.randomUUID().toString(),
            title = state.title,
            username = state.username,
            password = state.password,
            url = state.url,
            notes = state.notes,
            categoryId = state.selectedCategory!!.id,
            icon = null,
            isFavorite = state.isFavorite,
            createdAt = System.currentTimeMillis(),
            updatedAt = System.currentTimeMillis()
        )
        
        _state.value = _state.value.copy(isSaving = true)
        
        // Guardar
        createPasswordEntry(entry)
            .onSuccess {
                _state.value = _state.value.copy(isSaving = false)
                onNavigateBack()
            }
            .onFailure { error ->
                _state.value = _state.value.copy(
                    isSaving = false,
                    error = error.message ?: "Error al guardar"
                )
            }
    }
}
```

## Edición vs Creación

### Modo Creación
- Título: "Nueva Contraseña"
- Campos vacíos
- Categoría por defecto: "General"
- Al guardar: `CreatePasswordEntry`

### Modo Edición
- Título: "Editar Contraseña"
- Campos prellenados con datos existentes
- Al guardar: `UpdatePasswordEntry`
- Actualiza `updatedAt`

```kotlin
// Detectar modo de operación
val isEditing = entryId != null

// Cargar datos si es edición
LaunchedEffect(entryId) {
    if (entryId != null) {
        val entry = getPasswordById(entryId)
        entry?.let {
            _state.value = _state.value.copy(
                title = it.title,
                username = it.username,
                password = it.password,
                url = it.url ?: "",
                notes = it.notes ?: "",
                selectedCategory = categories.find { c -> c.id == it.categoryId },
                isFavorite = it.isFavorite
            )
        }
    }
}
```

## Diálogo de Generador de Contraseñas

```kotlin
@Composable
fun PasswordOptionsDialog(
    options: PasswordOptions,
    onOptionsChange: (PasswordOptions) -> Unit,
    onDismiss: () -> Unit
) {
    AlertDialog(
        onDismissRequest = onDismiss,
        title = { Text("Opciones de generación") },
        text = {
            Column {
                // Longitud
                Text("Longitud: ${options.length}")
                Slider(
                    value = options.length.toFloat(),
                    onValueChange = { 
                        onOptionsChange(options.copy(length = it.toInt()))
                    },
                    valueRange = 8f..128f,
                    steps = 120
                )
                
                // Opciones de caracteres
                RowCheckbox(
                    checked = options.includeUppercase,
                    onCheckedChange = {
                        onOptionsChange(options.copy(includeUppercase = it))
                    },
                    label = "Mayúsculas (A-Z)"
                )
                
                RowCheckbox(
                    checked = options.includeLowercase,
                    onCheckedChange = {
                        onOptionsChange(options.copy(includeLowercase = it))
                    },
                    label = "Minúsculas (a-z)"
                )
                
                RowCheckbox(
                    checked = options.includeNumbers,
                    onCheckedChange = {
                        onOptionsChange(options.copy(includeNumbers = it))
                    },
                    label = "Números (0-9)"
                )
                
                RowCheckbox(
                    checked = options.includeSymbols,
                    onCheckedChange = {
                        onOptionsChange(options.copy(includeSymbols = it))
                    },
                    label = "Símbolos (!@#$...)"
                )
            }
        },
        confirmButton = {
            TextButton(onClick = onDismiss) {
                Text("Aceptar")
            }
        }
    )
}
```

## PasswordOptions

```kotlin
data class PasswordOptions(
    val length: Int = 16,
    val includeUppercase: Boolean = true,
    val includeLowercase: Boolean = true,
    val includeNumbers: Boolean = true,
    val includeSymbols: Boolean = true
)
```

## Medidor de Fortaleza

```kotlin
@Composable
fun PasswordStrengthIndicator(password: String) {
    val strength = PasswordValidator.getStrengthScore(password)
    val color = when {
        strength < 30 -> MaterialTheme.colorScheme.error
        strength < 70 -> MaterialTheme.colorScheme.tertiary
        else -> MaterialTheme.colorScheme.primary
    }
    
    Column {
        LinearProgressIndicator(
            progress = strength / 100f,
            color = color,
            modifier = Modifier.fillMaxWidth()
        )
        Text(
            text = when {
                strength < 30 -> "Débil"
                strength < 70 -> "Media"
                else -> "Fuerte"
            },
            style = MaterialTheme.typography.bodySmall,
            color = color
        )
    }
}
```

## Casos de Uso Involucrados

| Caso de Uso | Propósito |
|-------------|-----------|
| `CreatePasswordEntry` | Crear nueva contraseña |
| `UpdatePasswordEntry` | Actualizar contraseña existente |
| `GetPasswordById` | Obtener contraseña para editar |
| `GetCategories` | Obtener lista de categorías |
| `GeneratePassword` | Generar contraseña segura |

## Testing

### UI Tests

```kotlin
@Test
fun passwordFormScreen_validInput_enablesSaveButton() {
    // Given
    composeTestRule.setContent {
        PasswordFormScreen(viewModel = fakeViewModel)
    }
    
    // When
    composeTestRule
        .onNodeWithLabel("Título")
        .performTextInput("Google")
    
    composeTestRule
        .onNodeWithLabel("Username")
        .performTextInput("user@gmail.com")
    
    composeTestRule
        .onNodeWithLabel("Contraseña")
        .performTextInput("SecurePass123!")
    
    // Then
    composeTestRule
        .onNodeWithText("Guardar")
        .assertIsEnabled()
}

@Test
fun passwordFormScreen_emptyTitle_showsError() {
    // Given
    composeTestRule.setContent {
        PasswordFormScreen(viewModel = fakeViewModel)
    }
    
    // When
    composeTestRule
        .onNodeWithLabel("Username")
        .performTextInput("user")
    
    composeTestRule
        .onNodeWithLabel("Contraseña")
        .performTextInput("pass123")
    
    composeTestRule
        .onNodeWithContentDescription("Guardar")
        .performClick()
    
    // Then
    composeTestRule
        .onNodeWithText("El título es requerido")
        .assertIsDisplayed()
}
```

### ViewModel Tests

```kotlin
@Test
fun `should create entry when all fields are valid`() = runTest {
    // Given
    val viewModel = PasswordFormViewModel(...)
    viewModel.onFieldChange(FormField.TITLE, "Google")
    viewModel.onFieldChange(FormField.USERNAME, "user@gmail.com")
    viewModel.onFieldChange(FormField.PASSWORD, "SecurePass123!")
    viewModel.onCategorySelected(Category(id = "cat-1", name = "General"))
    
    coEvery { createPasswordEntry(any()) } returns Result.success(Unit)
    
    // When
    viewModel.onSave()
    
    // Then
    coVerify { createPasswordEntry(match { it.title == "Google" }) }
}

@Test
fun `should fail validation when title is empty`() = runTest {
    // Given
    val viewModel = PasswordFormViewModel(...)
    viewModel.onFieldChange(FormField.USERNAME, "user")
    viewModel.onFieldChange(FormField.PASSWORD, "pass")
    
    // When
    viewModel.onSave()
    
    // Then
    val state = viewModel.state.first()
    assertEquals("El título es requerido", state.error)
}
```

## Accesibilidad

- **Labels**: Todos los TextField tienen label
- **ErrorMessages**: Se anuncian con `liveRegion = Assertive`
- **TouchTargets**: Mínimo 48x48dp
- **FocusOrder**: Orden lógico de tabulación

## Referencias

- [Material Design Text Fields](https://m3.material.io/components/text-fields)
- [Compose Forms](https://developer.android.com/jetpack/compose/forms)
- [Validation Patterns](https://m3.material.io/components/text-fields/guidelines)

---

**Documentación Relacionada:**
- [PasswordGeneratorScreen](password-generator.md)
- [PasswordListScreen](password-list.md)
- [CategoryPicker](../components/category-picker.md)
