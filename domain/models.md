# Modelos de Dominio

Los modelos de negocio representan las entidades fundamentales del dominio.

**Características clave**:
- ✅ Independiente de Android
- ✅ Independiente de frameworks
- ✅ Pure Kotlin
- ✅ Fácilmente testeable

---

## PasswordEntry

**Archivo**: `model/PasswordEntry.kt`

```kotlin
data class PasswordEntry(
    val id: String,
    val title: String,
    val username: String,
    val password: String,
    val url: String?,
    val notes: String?,
    val categoryId: String,
    val icon: String?,
    val isFavorite: Boolean,
    val createdAt: Long,
    val updatedAt: Long
)
```

### Campos

| Campo | Tipo | Descripción | Ejemplo |
|-------|------|-------------|---------|
| `id` | `String` | UUID único | `"550e8400-e29b-41d4-a716-446655440000"` |
| `title` | `String` | Título descriptivo | `"Cuenta de Google"` |
| `username` | `String` | Nombre de usuario o email | `"usuario@gmail.com"` |
| `password` | `String` | Contraseña (descifrada en dominio) | `"MiPass123!"` |
| `url` | `String?` | URL del servicio | `"https://google.com"` |
| `notes` | `String?` | Notas adicionales | `"Cuenta personal"` |
| `categoryId` | `String` | FK a Category | `"cat-123"` |
| `icon` | `String?` | Nombre del icono Font Awesome | `"FaLock"` |
| `isFavorite` | `Boolean` | Marcado como favorito | `true` |
| `createdAt` | `Long` | Timestamp creación (epoch ms) | `1718000000000` |
| `updatedAt` | `Long` | Timestamp actualización (epoch ms) | `1718000000000` |

### Reglas de Negocio

- `id`: UUID generado al crear
- `password`: Siempre cifrado cuando se persiste
- `notes`: Opcional, cifrado cuando existe
- `isFavorite`: Determina si aparece en sección de favoritos
- `createdAt/updatedAt`: Timestamps en milisegundos epoch

### Métodos de Extensión

```kotlin
// Verificar si la entrada tiene URL válida
fun PasswordEntry.hasValidUrl(): Boolean {
    return url?.isNotBlank() == true && 
           url?.startsWith("http") == true
}

// Obtener dominio de la URL
fun PasswordEntry.getDomain(): String? {
    return url?.let {
        try {
            URL(it).host
        } catch (e: Exception) {
            null
        }
    }
}

// Verificar si está vencida (más de 90 días sin actualizar)
fun PasswordEntry.isOld(thresholdDays: Int = 90): Boolean {
    val now = System.currentTimeMillis()
    val age = now - updatedAt
    return age > (thresholdDays.toLong() * 24 * 60 * 60 * 1000)
}
```

---

## Category

**Archivo**: `model/Category.kt`

```kotlin
@Serializable
data class Category(
    val id: String,
    val name: String,
    val color: Int,
    val icon: String,
    val isCustom: Boolean,
    val isDeletable: Boolean
)
```

### Campos

| Campo | Tipo | Descripción | Ejemplo |
|-------|------|-------------|---------|
| `id` | `String` | UUID único | `"cat-123"` |
| `name` | `String` | Nombre visible | `"Redes Sociales"` |
| `color` | `Int` | Color ARGB | `0xFF2196F3` (azul) |
| `icon` | `String` | Icono Font Awesome | `"FaUsers"` |
| `isCustom` | `Boolean` | true = creada por usuario | `false` (predefinida) |
| `isDeletable` | `Boolean` | false = no se puede eliminar | `false` (predefinida) |

### Tipos de Categorías

| Tipo | isCustom | isDeletable | Ejemplo |
|------|----------|-------------|---------|
| Predefinida | false | false | General, Finanzas |
| Personalizada | true | true | Categorías creadas por usuario |

### Categorías Predefinidas

```kotlin
val PREDEFINED_CATEGORIES = listOf(
    Category(id = "general", name = "General", color = 0xFF9E9E9E.toInt(), icon = "FaFolder", isCustom = false, isDeletable = false),
    Category(id = "social", name = "Redes Sociales", color = 0xFF2196F3.toInt(), icon = "FaUsers", isCustom = false, isDeletable = false),
    Category(id = "finance", name = "Finanzas", color = 0xFF4CAF50.toInt(), icon = "FaMoneyBill", isCustom = false, isDeletable = false),
    Category(id = "shopping", name = "Compras", color = 0xFFFF9800.toInt(), icon = "FaShoppingCart", isCustom = false, isDeletable = false),
    Category(id = "entertainment", name = "Entretenimiento", color = 0xFFF44336.toInt(), icon = "FaFilm", isCustom = false, isDeletable = false),
    Category(id = "work", name = "Trabajo", color = 0xFF9C27B0.toInt(), icon = "FaBriefcase", isCustom = false, isDeletable = false),
    Category(id = "education", name = "Educación", color = 0xFFFFEB3B.toInt(), icon = "FaGraduationCap", isCustom = false, isDeletable = false),
    Category(id = "health", name = "Salud", color = 0xFFFFC0CB.toInt(), icon = "FaHeartbeat", isCustom = false, isDeletable = false),
    Category(id = "travel", name = "Viajes", color = 0xFF00BCD4.toInt(), icon = "FaPlane", isCustom = false, isDeletable = false),
    Category(id = "others", name = "Otros", color = 0xFF795548.toInt(), icon = "FaArchive", isCustom = false, isDeletable = false)
)
```

### Métodos de Extensión

```kotlin
// Obtener color como Color de Compose
fun Category.getColor(): Color {
    return Color(color)
}

// Verificar si es categoría predefinida
fun Category.isPredefined(): Boolean = !isCustom
```

---

## AppSettings

**Archivo**: `model/AppSettings.kt`

```kotlin
data class AppSettings(
    val themeMode: ThemeMode = ThemeMode.AUTO,
    val isBiometricEnabled: Boolean = false,
    val lockTimeoutMs: Long = 300_000,
    val hasMasterPassword: Boolean = false
)

enum class ThemeMode(val value: Int) {
    AUTO(0),
    LIGHT(1),
    DARK(2)
}
```

### Campos

| Campo | Tipo | Default | Descripción |
|-------|------|---------|-------------|
| `themeMode` | `ThemeMode` | `AUTO` | Tema de la aplicación |
| `isBiometricEnabled` | `Boolean` | `false` | Biometría activada |
| `lockTimeoutMs` | `Long` | `300000` | Timeout de bloqueo (5 min) |
| `hasMasterPassword` | `Boolean` | `false` | Si hay contraseña configurada |

---

## BackupData

**Archivo**: `model/BackupData.kt`

Estructura para exportar/importar datos.

```kotlin
@Serializable
data class BackupData(
    val version: String = "1.0",
    val exportDate: Long = System.currentTimeMillis(),
    val entries: List<EncryptedPasswordEntry>,
    val categories: List<Category>
)

@Serializable
data class EncryptedPasswordEntry(
    val id: String,
    val title: String,
    val username: String,
    val encryptedPassword: String,  // Base64
    val url: String?,
    val encryptedNotes: String?,    // Base64
    val categoryId: String,
    val icon: String?,
    val isFavorite: Boolean,
    val createdAt: Long,
    val updatedAt: Long
)
```

### WeakPasswordEntry

Para auditoría de contraseñas débiles:

```kotlin
data class WeakPasswordEntry(
    val id: String,
    val title: String,
    val password: String,
    val weaknessReason: String,
    val strengthScore: Int
)
```
