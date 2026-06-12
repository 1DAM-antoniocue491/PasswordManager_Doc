# Base de Datos Room

## PasswordDatabase

La base de datos principal contiene tres tablas:

```kotlin
@Database(
    entities = [
        PasswordEntryEntity::class,
        CategoryEntity::class,
        SettingsEntity::class
    ],
    version = 1,
    exportSchema = false
)
@TypeConverters(DateConverters::class)
abstract class PasswordDatabase : RoomDatabase() {
    abstract fun passwordEntryDao(): PasswordEntryDao
    abstract fun categoryDao(): CategoryDao
    abstract fun settingsDao(): SettingsDao
    
    companion object {
        fun getInstance(context: Context): PasswordDatabase {
            return INSTANCE ?: synchronized(this) {
                Room.databaseBuilder(
                    context.applicationContext,
                    PasswordDatabase::class.java,
                    "passwords.db"
                ).build()
            }
        }
    }
}
```

**Características**:
- **Nombre**: `passwords.db`
- **Versión**: 1
- **TypeConverters**: `DateConverters` para manejar timestamps

---

## Entidades

### PasswordEntryEntity

Representa una entrada de contraseña en la base de datos.

```kotlin
@Entity(tableName = "password_entries")
data class PasswordEntryEntity(
    @PrimaryKey val id: String,
    val title: String,
    val username: String,
    val password: String,        // Cifrado (Base64)
    val url: String?,
    val notes: String?,          // Cifrado (Base64)
    val categoryId: String,      // FK a categories
    val icon: String?,
    val isFavorite: Boolean,
    val createdAt: Long,         // Timestamp epoch
    val updatedAt: Long          // Timestamp epoch
)
```

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | String | UUID único, Primary Key |
| `title` | String | Título descriptivo |
| `username` | String | Nombre de usuario |
| `password` | String | Contraseña cifrada en Base64 |
| `url` | String? | URL del sitio (opcional) |
| `notes` | String? | Notas adicionales cifradas |
| `categoryId` | String | FK a `categories(id)` |
| `icon` | String? | Nombre del icono (ej: `FaLock`) |
| `isFavorite` | Boolean | Marcado como favorito |
| `createdAt` | Long | Timestamp de creación (epoch ms) |
| `updatedAt` | Long | Timestamp de actualización (epoch ms) |

**Índices implícitos**:
- Primary Key en `id`
- Foreign Key implícita en `categoryId`

---

### CategoryEntity

Representa una categoría para organizar contraseñas.

```kotlin
@Entity(tableName = "categories")
data class CategoryEntity(
    @PrimaryKey val id: String,
    val name: String,
    val color: Int,              // Color ARGB
    val icon: String,
    val isCustom: Boolean,
    val isDeletable: Boolean
)
```

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | String | UUID único |
| `name` | String | Nombre de la categoría |
| `color` | Int | Color ARGB (ej: 0xFFFF5722) |
| `icon` | String | Icono de Font Awesome |
| `isCustom` | Boolean | true = creada por usuario |
| `isDeletable` | Boolean | false = predefinida (no se puede eliminar) |

**Categorías Predefinidas**:
- General (gris)
- Redes Sociales (azul)
- Finanzas (verde)
- Compras (naranja)
- Entretenimiento (rojo)
- Trabajo (morado)
- Educación (amarillo)
- Salud (rosa)
- Viajes (cyan)
- Otros (marrón)

---

### SettingsEntity

Almacena configuración clave-valor.

```kotlin
@Entity(tableName = "settings")
data class SettingsEntity(
    @PrimaryKey val key: String,
    val value: String,
    val updatedAt: Long
)
```

**Claves utilizadas**:

| Key | Value Type | Descripción |
|-----|------------|-------------|
| `theme_mode` | Int | 0=Auto, 1=Light, 2=Dark |
| `biometric_enabled` | Boolean | Si biometría está activa |
| `lock_timeout` | Long | Timeout en milisegundos |

---

## TypeConverters

### DateConverters

Convierte entre `Long` (timestamp) y `Date` para Room.

```kotlin
class DateConverters {
    @TypeConverter
    fun fromTimestamp(value: Long): Date {
        return Date(value)
    }
    
    @TypeConverter
    fun dateToTimestamp(date: Date): Long {
        return date.time
    }
}
```

**Nota**: Las entidades usan `Long` directamente, pero este converter permite usar `Date` si es necesario.

---

## Migraciones de Base de Datos

Actualmente la base de datos está en versión 1. Para futuras migraciones:

```kotlin
Room.databaseBuilder(
    context,
    PasswordDatabase::class.java,
    "passwords.db"
)
.addMigrations(object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        // Agregar nueva columna
        database.execSQL("ALTER TABLE password_entries ADD COLUMN lastUsedAt INTEGER")
    }
})
.build()
```

---

## Consideraciones de Rendimiento

### Índices

Para mejorar rendimiento en búsquedas:

```kotlin
@Entity(
    tableName = "password_entries",
    indices = [
        Index(value = ["title"]),
        Index(value = ["categoryId"]),
        Index(value = ["isFavorite"])
    ]
)
data class PasswordEntryEntity(...)
```

### Consultas Optimizadas

```kotlin
// ✅ BUENO: Usa índice en categoryId
@Query("SELECT * FROM password_entries WHERE categoryId = :categoryId")
fun getByCategory(categoryId: String): Flow<List<PasswordEntryEntity>>

// ❌ MALO: No usa índice, escaneo completo
@Query("SELECT * FROM password_entries WHERE title LIKE '%' || :query || '%'")
fun searchSlow(query: String): Flow<List<PasswordEntryEntity>>

// ✅ MEJOR: LIKE con wildcard al final usa índice
@Query("SELECT * FROM password_entries WHERE title LIKE :query || '%'")
fun searchOptimized(query: String): Flow<List<PasswordEntryEntity>>
```
