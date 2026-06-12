# Pantallas (Screens)

Documentación completa de todas las pantallas de la aplicación.

---

# PasswordListScreen

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

# LoginScreen

## Descripción

Pantalla de autenticación inicial donde el usuario ingresa su contraseña maestra o usa biometría.

**Archivo**: `LoginScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│                                         │
│           🔐 Password Manager           │
│                                         │
│         Contraseña Maestra              │
│  ┌─────────────────────────────────┐   │
│  │ ••••••••••••••••••        👁️    │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │         🔓 Entrar               │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │         👆 Huella/Rostro        │   │
│  └─────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

## Estado (State)

```kotlin
data class LoginState(
    val password: String = "",
    val isLoading: Boolean = false,
    val error: String? = null,
    val isBiometricAvailable: Boolean = false,
    val hasMasterPassword: Boolean = false
)
```

## Eventos de UI

| Evento | Parámetros | Descripción |
|--------|------------|-------------|
| `onPasswordChange` | `password: String` | Usuario escribe contraseña |
| `onLoginClick` | - | Iniciar sesión con contraseña |
| `onBiometricClick` | - | Autenticar con biometría |
| `onTogglePasswordVisibility` | - | Mostrar/ocultar contraseña |
| `clearError` | - | Limpiar mensaje de error |

## Flujo de Autenticación

### Con Contraseña

```
Usuario escribe contraseña
         ↓
onPasswordChange → Actualiza state.password
         ↓
Usuario pulsa "Entrar"
         ↓
ViewModel.authenticateUser(password)
         ↓
UseCase → AuthRepository.authenticateUser()
         ↓
1. Derivar clave con PBKDF2
2. Descifrar clave almacenada (RSA + biometría)
3. Comparar claves
         ↓
¿Coinciden? ──No──> Mostrar error
     │
     Sí
     ↓
Guardar clave en memoria
     ↓
Navegar a Home
```

### Con Biometría

```
Usuario pulsa "Huella/Rostro"
         ↓
BiometricPrompt.show()
         ↓
Usuario autentica (huella/rostro)
         ↓
¿Éxito? ──No──> Mostrar error
     │
     Sí
     ↓
Keystore desbloquea clave privada
     ↓
Descifrar clave maestra
     ↓
Navegar a Home
```

## Validaciones

- Contraseña no vacía para habilitar botón "Entrar"
- Biometría solo disponible si está configurada
- Timeout de intento después de 5 fallos

## Casos de Uso Involucrados

| Caso de Uso | Propósito |
|-------------|-----------|
| `AuthenticateUser` | Autenticar con contraseña maestra |
| `SetupMasterPassword` | Configurar contraseña inicial (onboarding) |
| `HasMasterPassword` | Verificar si ya está configurada |

---

# HomeScreen

## Descripción

Pantalla principal de inicio que muestra el menú de navegación a todas las funcionalidades de la app.

**Archivo**: `HomeScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  Password Manager             👤 🔔     │
├─────────────────────────────────────────┤
│                                         │
│  ┌───────────┐    ┌───────────┐        │
│  │   🔐      │    │   🔑      │        │
│  │Contraseñas│    │Generador  │        │
│  └───────────┘    └───────────┘        │
│                                         │
│  ┌───────────┐    ┌───────────┐        │
│  │   📁      │    │   💾      │        │
│  │Categorías │    │  Backup   │        │
│  └───────────┘    └───────────┘        │
│                                         │
│  ┌───────────┐    ┌───────────┐        │
│  │   🛡️      │    │   📊      │        │
│  │ Auditoría │    │Estadísticas│       │
│  └───────────┘    └───────────┘        │
│                                         │
│  ┌─────────────────────────────┐       │
│  │   ⚙️  Configuración         │       │
│  └─────────────────────────────┘       │
│                                         │
└─────────────────────────────────────────┘
```

## Opciones de Navegación

| Opción | Icono | Destino | Descripción |
|--------|-------|---------|-------------|
| Contraseñas | `FaLock` | `PasswordListScreen` | Lista de contraseñas |
| Generador | `FaKey` | `PasswordGeneratorScreen` | Generar contraseñas |
| Categorías | `FaFolders` | `CategoryManagementScreen` | Gestionar categorías |
| Backup | `FaDownload` | `BackupScreen` | Exportar/Importar |
| Auditoría | `FaShield` | `AuditScreen` | Revisar seguridad |
| Estadísticas | `FaChartBar` | `StatisticsScreen` | Ver estadísticas |
| Configuración | `FaCog` | `SettingsScreen` | Ajustes de la app |

## Estado de Sesión

Muestra en la cabecera:
- Icono de usuario
- Notificaciones (si hay contraseñas débiles)
- Tiempo restante para bloqueo automático

## Eventos de UI

| Evento | Destino |
|--------|---------|
| `onNavigateToPasswordList` | `PasswordListScreen` |
| `onNavigateToGenerator` | `PasswordGeneratorScreen` |
| `onNavigateToCategories` | `CategoryManagementScreen` |
| `onNavigateToBackup` | `BackupScreen` |
| `onNavigateToAudit` | `AuditScreen` |
| `onNavigateToStatistics` | `StatisticsScreen` |
| `onNavigateToSettings` | `SettingsScreen` |

---

# PasswordDetailScreen

## Descripción

Muestra el detalle completo de una contraseña específica con opciones de copiar, editar y eliminar.

**Archivo**: `PasswordDetailScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  ← Google                       ✏️ 🗑️  │
├─────────────────────────────────────────┤
│                                         │
│           🔐                            │
│                                         │
│  Título                                 │
│  Google                                 │
│                                         │
│  Username                               │
│  usuario@gmail.com              [📋]   │
│                                         │
│  Contraseña                             │
│  ••••••••••••••••••            [📋][👁] │
│                                         │
│  URL                                    │
│  https://google.com             [🔗]   │
│                                         │
│  Notas                                  │
│  Cuenta personal creada en 2020...      │
│                                         │
│  Categoría                              │
│  🔵 Redes Sociales                      │
│                                         │
│  Creado: 01/01/2024                     │
│  Actualizado: 15/06/2024                │
│                                         │
│  □ Favorito                             │
│                                         │
└─────────────────────────────────────────┘
```

## Estado (State)

```kotlin
data class PasswordDetailState(
    val entry: PasswordEntry? = null,
    val category: Category? = null,
    val isLoading: Boolean = true,
    val showPassword: Boolean = false,
    val error: String? = null
)
```

## Funcionalidades

### Copiar al Portapapeles

| Campo | Acción |
|-------|--------|
| Username | Botón 📋 copia el username |
| Contraseña | Botón 📋 copia la contraseña |
| URL | Botón 🔗 abre en navegador |

### Mostrar/Ocultar Contraseña

```kotlin
IconButton(onClick = { showPassword = !showPassword }) {
    Icon(
        imageVector = if (showPassword) Icons.Default.Visibility 
                      else Icons.Default.VisibilityOff,
        contentDescription = if (showPassword) "Ocultar" else "Mostrar"
    )
}
```

### Menú de Acciones

- **Editar** (✏️): Navega a `PasswordFormScreen` con el ID
- **Eliminar** (🗑️): Muestra diálogo de confirmación

## Casos de Uso Involucrados

| Caso de Uso | Propósito |
|-------------|-----------|
| `GetPasswordById` | Obtener contraseña por ID |
| `GetCategories` | Obtener categoría para mostrar color/nombre |

---

# PasswordGeneratorScreen

## Descripción

Generador de contraseñas seguras con opciones configurables y medidor de fortaleza.

**Archivo**: `PasswordGeneratorScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  ← Generador de Contraseñas             │
├─────────────────────────────────────────┤
│                                         │
│  ┌─────────────────────────────────┐   │
│  │  K#9mX$2pL@5nQr8Y        [📋]   │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Fortaleza: [████████░░] Fuerte         │
│                                         │
│  Longitud: 16                           │
│  ├────────────●────────────┤ 8-128      │
│                                         │
│  Opciones                               │
│  ┌─────────────────────────────────┐   │
│  │ ☑ Mayúsculas (A-Z)              │   │
│  │ ☑ Minúsculas (a-z)              │   │
│  │ ☑ Números (0-9)                 │   │
│  │ ☑ Símbolos (!@#$...)            │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │      🔄 Generar Nueva           │   │
│  └─────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

## Estado (State)

```kotlin
data class PasswordGeneratorState(
    val generatedPassword: String = "",
    val length: Int = 16,
    val includeUppercase: Boolean = true,
    val includeLowercase: Boolean = true,
    val includeNumbers: Boolean = true,
    val includeSymbols: Boolean = true,
    val strength: PasswordStrength = PasswordStrength.MEDIUM
)

enum class PasswordStrength {
    WEAK, MEDIUM, STRONG
}
```

## Algoritmo de Generación

```kotlin
fun generatePassword(options: PasswordOptions): String {
    val charPool = buildString {
        if (options.includeUppercase) append('A'..'Z')
        if (options.includeLowercase) append('a'..'z')
        if (options.includeNumbers) append('0'..'9')
        if (options.includeSymbols) append("!@#$%^&*()_+-=[]{}|;:,.<>?")
    }
    
    require(charPool.isNotEmpty()) { "Debe seleccionar al menos un tipo de carácter" }
    
    return (1..options.length)
        .map { SecureRandom().nextInt(charPool.size) }
        .map(charPool::get)
        .joinToString("")
}
```

## Medidor de Fortaleza

| Criterio | Puntos |
|----------|--------|
| Longitud ≥ 12 | +30 |
| Longitud ≥ 16 | +20 |
| Tiene mayúsculas | +15 |
| Tiene minúsculas | +15 |
| Tiene números | +15 |
| Tiene símbolos | +20 |
| Sin patrones repetidos | +10 |

**Resultado**:
- 0-40: Débil (rojo)
- 41-70: Media (amarillo)
- 71-100: Fuerte (verde)

## Casos de Uso Involucrados

| Caso de Uso | Propósito |
|-------------|-----------|
| `GeneratePassword` | Generar contraseña aleatoria |

---

# CategoryManagementScreen

## Descripción

Gestión de categorías personalizadas: crear, editar, eliminar y organizar categorías.

**Archivo**: `CategoryManagementScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  ← Categorías                  ➕       │
├─────────────────────────────────────────┤
│                                         │
│  📁 General                    (gris)   │
│  📁 Redes Sociales             (azul)   │
│  📁 Finanzas                   (verde)  │
│  📁 Compras                    (naranja)│
│  📁 Entretenimiento            (rojo)   │
│  📁 Trabajo                    (morado) │
│  📁 Educación                  (amarillo)│
│  📁 Salud                      (rosa)   │
│  📁 Viajes                     (cyan)   │
│  📁 Otros                      (marrón) │
│  ─────────────────────────────────      │
│  ✏️ Mis Categorías                      │
│  📁 Categoría Personal 1       (color)  │
│  📁 Categoría Personal 2       (color)  │
│                                         │
└─────────────────────────────────────────┘
```

## Diálogo de Creación/Edición

```
┌───────────────────────────────────┐
│  Nueva Categoría            [X]  │
├───────────────────────────────────┤
│  Nombre                           │
│  ┌─────────────────────────────┐ │
│  │ Mi Categoría                │ │
│  └─────────────────────────────┘ │
│                                   │
│  Color                            │
│  🔴 🩷 🟣 🟙 🟠 🔵 🔷 🔶         │
│  🟢 🟩 🟨 🟧 🟤 ⚫ ⚪ 🟫         │
│                                   │
│  Icono                            │
│  ┌─────────────────────────────┐ │
│  │ 📁 FaFolder             ▼  │ │
│  └─────────────────────────────┘ │
│                                   │
│       [Cancelar] [Guardar]        │
└───────────────────────────────────┘
```

## Estado (State)

```kotlin
data class CategoryManagementState(
    val categories: List<Category> = emptyList(),
    val isLoading: Boolean = true,
    val error: String? = null,
    val showCreateDialog: Boolean = false,
    val editingCategory: Category? = null,
    val categoryName: String = "",
    val selectedColor: Int = 0xFF9E9E9E.toInt(),
    val selectedIcon: String = "FaFolder"
)
```

## Reglas de Negocio

### Categorías Predefinidas
- No se pueden eliminar (`isDeletable = false`)
- No se puede modificar `isDeletable`
- Nombre e icono fijos

### Categorías Personalizadas
- Se pueden eliminar (`isDeletable = true`)
- Al eliminar, las contraseñas se mueven a "General"
- Se pueden editar nombre, color e icono

## Eventos de UI

| Evento | Parámetros | Descripción |
|--------|------------|-------------|
| `onCreateCategory` | `name, color, icon` | Crear nueva categoría |
| `onEditCategory` | `category` | Abrir diálogo de edición |
| `onDeleteCategory` | `category` | Eliminar categoría personalizada |
| `onUpdateCategory` | `category` | Guardar cambios |

## Casos de Uso Involucrados

| Caso de Uso | Propósito |
|-------------|-----------|
| `GetCategories` | Obtener todas las categorías |
| `CreateCategory` | Crear categoría personalizada |
| `UpdateCategory` | Actualizar categoría |
| `DeleteCategory` | Eliminar categoría personalizada |

---

# SettingsScreen

## Descripción

Configuración general de la aplicación: tema, seguridad, backup y datos.

**Archivo**: `SettingsScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  ← Configuración                        │
├─────────────────────────────────────────┤
│                                         │
│  APARIENCIA                             │
│  ┌─────────────────────────────────┐   │
│  │ Tema                            │   │
│  │ Automático              →       │   │
│  └─────────────────────────────────┘   │
│                                         │
│  SEGURIDAD                              │
│  ┌─────────────────────────────────┐   │
│  │ Autenticación Biométrica   ☑   │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │ Bloqueo Automático              │   │
│  │ 5 minutos               →       │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │ Cambiar Contraseña Maestra  →   │   │
│  └─────────────────────────────────┘   │
│                                         │
│  DATOS                                  │
│  ┌─────────────────────────────────┐   │
│  │ Exportar Backup             →   │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │ Importar desde CSV          →   │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │ Borrar Todos Datos      ⚠️  →   │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ACERCA DE                              │
│  ┌─────────────────────────────────┐   │
│  │ Versión: 1.0                →   │   │
│  └─────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

## Opciones de Configuración

### Apariencia

| Opción | Valores | Default |
|--------|---------|---------|
| Tema | Claro / Oscuro / Automático | Automático |

### Seguridad

| Opción | Tipo | Descripción |
|--------|------|-------------|
| Biometría | Toggle | Activar/desactivar huella/rostro |
| Bloqueo Automático | Selector | 1, 2, 5, 10 minutos, o nunca |
| Cambiar Contraseña | Navegación | Abre `ChangePasswordScreen` |

### Datos

| Opción | Acción |
|--------|--------|
| Exportar Backup | Guarda archivo cifrado en almacenamiento |
| Importar CSV | Lee archivo CSV y importa contraseñas |
| Borrar Datos | Elimina toda la BD (requiere confirmación) |

## Estado (State)

```kotlin
data class SettingsState(
    val themeMode: ThemeMode = ThemeMode.AUTO,
    val isBiometricEnabled: Boolean = false,
    val lockTimeoutMs: Long = 300_000,
    val hasMasterPassword: Boolean = true,
    val isExporting: Boolean = false,
    val isImporting: Boolean = false,
    val error: String? = null
)

enum class ThemeMode(val value: Int) {
    AUTO(0),
    LIGHT(1),
    DARK(2)
}
```

## Casos de Uso Involucrados

| Caso de Uso | Propósito |
|-------------|-----------|
| `GetThemeMode` / `SetThemeMode` | Gestionar tema |
| `IsBiometricEnabled` / `SetBiometricEnabled` | Biometría |
| `GetLockTimeout` / `SetLockTimeout` | Timeout de bloqueo |
| `ExportEncryptedBackup` | Exportar datos |
| `ImportFromCSV` | Importar datos |

---

# BackupScreen

## Descripción

Gestión de copias de seguridad: exportar datos cifrados e importar desde CSV.

**Archivo**: `BackupScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  ← Copia de Seguridad                   │
├─────────────────────────────────────────┤
│                                         │
│  ┌─────────────────────────────────┐   │
│  │  📤 EXPORTAR                    │   │
│  │                                 │   │
│  │  Guarda una copia cifrada de    │   │
│  │  todas tus contraseñas en un    │   │
│  │  archivo .backup                │   │
│  │                                 │   │
│  │  ┌───────────────────────────┐  │   │
│  │  │   Seleccionar Ubicación   │  │   │
│  │  └───────────────────────────┘  │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │  📥 IMPORTAR                    │   │
│  │                                 │   │
│  │  Importa contraseñas desde un   │   │
│  │  archivo CSV                    │   │
│  │                                 │   │
│  │  ┌───────────────────────────┐  │   │
│  │  │   Seleccionar Archivo     │  │   │
│  │  └───────────────────────────┘  │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │  ⚠️ ZONA DE PELIGRO             │   │
│  │                                 │   │
│  │  ┌───────────────────────────┐  │   │
│  │  │  Borrar Backup y Reset    │  │   │
│  │  └───────────────────────────┘  │   │
│  └─────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

## Exportar Backup

### Proceso

```
Usuario selecciona ubicación
         ↓
ViewModel.exportBackup(path)
         ↓
UseCase: ExportEncryptedBackup
         ↓
1. Obtener todas las contraseñas
2. Obtener categorías
3. Cifrar datos con clave de backup
4. Serializar a JSON
5. Guardar en archivo
         ↓
¿Éxito? ──No──> Mostrar error
     │
     Sí
     ↓
Mostrar confirmación
```

### Formato de Archivo

```json
{
  "version": "1.0",
  "exportDate": 1718000000000,
  "entries": [
    {
      "id": "uuid",
      "title": "Google",
      "username": "encrypted...",
      "encryptedPassword": "base64...",
      "url": "https://google.com",
      "encryptedNotes": "base64...",
      "categoryId": "uuid",
      "isFavorite": false,
      "createdAt": 1718000000000,
      "updatedAt": 1718000000000
    }
  ],
  "categories": [...]
}
```

## Importar CSV

### Formato CSV Esperado

```csv
title,username,password,url,notes
Google,user@gmail.com,SecurePass123,https://google.com,Cuenta personal
Facebook,user@fb.com,AnotherPass!,https://facebook.com,
```

### Proceso

```
Usuario selecciona archivo CSV
         ↓
ViewModel.importCsv(path)
         ↓
UseCase: ImportFromCSV
         ↓
1. Leer archivo línea por línea
2. Parsear columnas (title, username, password, url)
3. Crear PasswordEntry para cada línea
4. Insertar en base de datos
         ↓
¿Éxito? ──No──> Mostrar error
     │
     Sí
     ↓
Mostrar resumen: "X contraseñas importadas"
```

## Estado (State)

```kotlin
data class BackupState(
    val isExporting: Boolean = false,
    val isImporting: Boolean = false,
    val exportPath: String? = null,
    val importCount: Int? = null,
    val error: String? = null,
    val showConfirmReset: Boolean = false
)
```

## Casos de Uso Involucrados

| Caso de Uso | Propósito |
|-------------|-----------|
| `ExportEncryptedBackup` | Exportar datos cifrados |
| `ImportFromCSV` | Importar desde CSV |

---

# AuditScreen

## Descripción

Auditoría de seguridad que detecta contraseñas débiles, reutilizadas y patrones comunes.

**Archivo**: `AuditScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  ← Auditoría de Seguridad               │
├─────────────────────────────────────────┤
│                                         │
│  ┌─────────────────────────────────┐   │
│  │  Puntuación de Seguridad        │   │
│  │                                 │   │
│  │       [████████░░] 75/100       │   │
│  │         BUENA                   │   │
│  └─────────────────────────────────┘   │
│                                         │
│  RESUMEN                                │
│  ┌─────────────┐ ┌─────────────┐       │
│  │    12       │ │     3       │       │
│  │  Total      │ │  Débiles    │       │
│  └─────────────┘ └─────────────┘       │
│  ┌─────────────┐ ┌─────────────┐       │
│  │     2       │ │     5       │       │
│  │ Reutilizadas│ │ Antiguas    │       │
│  └─────────────┘ └─────────────┘       │
│                                         │
│  CONTRASEÑAS DÉBILES                    │
│  ┌─────────────────────────────────┐   │
│  │ ⚠️  Facebook                    │   │
│  │    Menos de 8 caracteres        │   │
│  │                         [Editar]│   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │ ⚠️  Netflix                     │   │
│  │    Sin números ni símbolos      │   │
│  │                         [Editar]│   │
│  └─────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

## Criterios de Auditoría

### Contraseña Débil

| Criterio | Threshold |
|----------|-----------|
| Longitud | < 8 caracteres |
| Complejidad | Sin mayúsculas |
| Complejidad | Sin números |
| Complejidad | Sin símbolos |
| Patrones | 123456, password, qwerty |

### Contraseña Reutilizada

- Mismo password en múltiples entradas
- Se detecta comparando hashes

### Contraseña Antigua

- Más de 90 días sin actualizar
- Se calcula desde `updatedAt`

## Estado (State)

```kotlin
data class AuditState(
    val totalPasswords: Int = 0,
    val weakPasswords: List<WeakPasswordEntry> = emptyList(),
    val reusedPasswords: List<ReusedPasswordEntry> = emptyList(),
    val oldPasswords: List<OldPasswordEntry> = emptyList(),
    val securityScore: Int = 0,
    val isLoading: Boolean = true,
    val error: String? = null
)

data class WeakPasswordEntry(
    val id: String,
    val title: String,
    val reason: String
)
```

## Cálculo de Puntuación

```kotlin
fun calculateSecurityScore(
    total: Int,
    weak: Int,
    reused: Int,
    old: Int
): Int {
    if (total == 0) return 0
    
    val weakRatio = weak.toFloat() / total
    val reusedRatio = reused.toFloat() / total
    val oldRatio = old.toFloat() / total
    
    val score = 100 * (1 - (weakRatio * 0.4 + reusedRatio * 0.3 + oldRatio * 0.3))
    return score.coerceIn(0, 100)
}
```

## Casos de Uso Involucrados

| Caso de Uso | Propósito |
|-------------|-----------|
| `AuditWeakPasswords` | Detectar contraseñas débiles |
| `GetSecurityStatistics` | Obtener estadísticas completas |

---

# StatisticsScreen

## Descripción

Estadísticas y gráficos sobre el uso de la aplicación y seguridad de contraseñas.

**Archivo**: `StatisticsScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  ← Estadísticas                         │
├─────────────────────────────────────────┤
│                                         │
│  RESUMEN                                │
│  ┌─────────────────────────────────┐   │
│  │  📊 Total Contraseñas: 24       │   │
│  │  ⭐ Favoritos: 8                │   │
│  │  📁 Categorías: 10              │   │
│  │  🛡️ Fortaleza Promedio: 75%    │   │
│  └─────────────────────────────────┘   │
│                                         │
│  CONTRASEÑAS POR CATEGORÍA              │
│  ┌─────────────────────────────────┐   │
│  │  ████████░░░░░░░░░░░░  General  │   │
│  │  ██████░░░░░░░░░░░░░░  Redes    │   │
│  │  ████░░░░░░░░░░░░░░░░  Finanzas │   │
│  │  ██░░░░░░░░░░░░░░░░░░  Compras  │   │
│  │  ...                            │   │
│  └─────────────────────────────────┘   │
│                                         │
│  FORTALEZA POR TIPO                     │
│  ┌─────────────────────────────────┐   │
│  │  Fuertes:   ████████░░  18      │   │
│  │  Medias:    ██░░░░░░░░   4      │   │
│  │  Débiles:   ░░░░░░░░░░   2      │   │
│  └─────────────────────────────────┘   │
│                                         │
│  HISTORIAL (últimos 30 días)            │
│  ┌─────────────────────────────────┐   │
│  │  📈 Gráfico de barras           │   │
│  │  Contraseñas añadidas por día   │   │
│  └─────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

## Estadísticas Mostradas

### Resumen

| Métrica | Fuente |
|---------|--------|
| Total Contraseñas | `COUNT(*) FROM password_entries` |
| Favoritos | `COUNT(*) WHERE isFavorite = 1` |
| Categorías | `COUNT(*) FROM categories` |
| Fortaleza Promedio | Media de `PasswordValidator.getStrengthScore()` |

### Por Categoría

```kotlin
data class CategoryStats(
    val categoryId: String,
    val categoryName: String,
    val color: Int,
    val count: Int,
    val percentage: Float
)
```

### Por Fortaleza

| Categoría | Rango |
|-----------|-------|
| Fuertes | 71-100 |
| Medias | 41-70 |
| Débiles | 0-40 |

## Estado (State)

```kotlin
data class StatisticsState(
    val totalPasswords: Int = 0,
    val favoriteCount: Int = 0,
    val categoryCount: Int = 0,
    val averageStrength: Float = 0f,
    val passwordsByCategory: List<CategoryStats> = emptyList(),
    val passwordsByStrength: StrengthDistribution = StrengthDistribution(),
    val isLoading: Boolean = true,
    val error: String? = null
)

data class StrengthDistribution(
    val strong: Int = 0,
    val medium: Int = 0,
    val weak: Int = 0
)
```

## Casos de Uso Involucrados

| Caso de Uso | Propósito |
|-------------|-----------|
| `GetSecurityStatistics` | Obtener todas las estadísticas |
| `GetAllPasswords` | Calcular métricas adicionales |
| `GetCategories` | Nombres de categorías para stats |

---

# OnboardingScreen

## Descripción

Pantalla de bienvenida para configurar la aplicación en primer uso.

**Archivo**: `OnboardingScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│                                         │
│           🔐                            │
│                                         │
│     Bienvenido a Password Manager       │
│                                         │
│  Tu gestor de contraseñas seguro y     │
│  privado. Todo se cifra en tu          │
│  dispositivo.                          │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │  [1/3]                          │   │
│  │  ██████░░░░░░░░░░░░░░░░         │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │      Comenzar →                 │   │
│  └─────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

## Pasos del Onboarding

### Paso 1: Bienvenida
- Explicación de la aplicación
- Características principales

### Paso 2: Configurar Contraseña Maestra
```
┌───────────────────────────────────┐
│  Contraseña Maestra               │
│                                   │
│  Elige una contraseña fuerte      │
│  que recordarás. No hay           │
│  recuperación si la olvidas.      │
│                                   │
│  ┌─────────────────────────────┐ │
│  │ ••••••••••••••••••    👁️   │ │
│  └─────────────────────────────┘ │
│                                   │
│  ┌─────────────────────────────┐ │
│  │ ••••••••••••••••••    👁️   │ │
│  │   (Confirmar)               │ │
│  └─────────────────────────────┘ │
│                                   │
│  Fortaleza: [████████░░] Fuerte   │
│                                   │
│       [Continuar →]               │
└───────────────────────────────────┘
```

### Paso 3: Biometría (Opcional)
```
┌───────────────────────────────────┐
│  Autenticación Biométrica         │
│                                   │
│  👆                               │
│                                   │
│  ¿Quieres usar tu huella o        │
│  rostro para desbloquear?         │
│                                   │
│  Puedes cambiar esto después      │
│  en Configuración.                │
│                                   │
│  [Omitir]     [Configurar →]      │
└───────────────────────────────────┘
```

## Validaciones

### Contraseña Maestra
- Mínimo 8 caracteres
- Al menos una mayúscula
- Al menos un número
- Al menos un símbolo
- Confirmación debe coincidir

## Estado (State)

```kotlin
data class OnboardingState(
    val currentStep: Int = 0,
    val masterPassword: String = "",
    val confirmPassword: String = "",
    val passwordStrength: Int = 0,
    val isBiometricAvailable: Boolean = false,
    val error: String? = null
)
```

## Casos de Uso Involucrados

| Caso de Uso | Propósito |
|-------------|-----------|
| `SetupMasterPassword` | Configurar contraseña inicial |
| `SeedPredefinedCategories` | Crear categorías por defecto |
| `SetBiometricEnabled` | Configurar biometría (opcional) |

---

# ChangePasswordScreen

## Descripción

Pantalla para cambiar la contraseña maestra del usuario.

**Archivo**: `ChangePasswordScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  ← Cambiar Contraseña Maestra           │
├─────────────────────────────────────────┤
│                                         │
│  Contraseña Actual                      │
│  ┌─────────────────────────────────┐   │
│  │ ••••••••••••••••••        👁️   │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Nueva Contraseña                       │
│  ┌─────────────────────────────────┐   │
│  │ ••••••••••••••••••        👁️   │   │
│  └─────────────────────────────────┘   │
│  Fortaleza: [████████░░] Fuerte        │
│                                         │
│  Confirmar Nueva Contraseña             │
│  ┌─────────────────────────────────┐   │
│  │ ••••••••••••••••••        👁️   │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ⚠️ Todas tus contraseñas se           │
│  volverán a cifrar con la nueva        │
│  clave. Este proceso no se puede       │
│  deshacer.                             │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │      Cambiar Contraseña         │   │
│  └─────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

## Estado (State)

```kotlin
data class ChangePasswordState(
    val currentPassword: String = "",
    val newPassword: String = "",
    val confirmPassword: String = "",
    val isChanging: Boolean = false,
    val error: String? = null,
    val success: Boolean = false
)
```

## Proceso de Cambio

```
Usuario ingresa contraseñas
         ↓
Validar formularios
         ↓
¿Válido? ──No──> Mostrar error
     │
     Sí
     ↓
ViewModel.changePassword()
         ↓
UseCase: ChangeMasterPassword
         ↓
1. Verificar contraseña actual
2. Obtener TODAS las contraseñas
3. Descifrar todas con clave vieja
4. Configurar nueva contraseña maestra
5. Re-cifrar todas con nueva clave
         ↓
¿Éxito? ──No──> Mostrar error / Revertir
     │
     Sí
     ↓
Mostrar éxito
     ↓
Navegar a Login (re-autenticar)
```

## Validaciones

| Campo | Validación |
|-------|------------|
| Contraseña Actual | No vacía, debe ser correcta |
| Nueva Contraseña | Mínimo 8 caracteres, complejidad |
| Confirmar | Debe coincidir con Nueva |

## Casos de Uso Involucrados

| Caso de Uso | Propósito |
|-------------|-----------|
| `ChangeMasterPassword` | Cambiar contraseña maestra |
| `AuthenticateUser` | Verificar contraseña actual |
| `GetAllPasswords` | Obtener contraseñas para re-cifrar |

---

**Documentación Relacionada:**
- [PasswordFormScreen](password-form.md)
- [ViewModels](../viewmodels/overview.md)
- [Navigation](../presentation/navigation.md)

**Documentación Relacionada:**
- [PasswordDetailScreen](password-detail.md)
- [PasswordFormScreen](password-form.md)
- [CategoryFilterChips](../components/category-filter-chips.md)
