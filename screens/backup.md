# BackupScreen

## Visión General

Pantalla para exportar e importar copias de seguridad de las contraseñas.

**Archivo**: `presentation/ui/screens/BackupScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  ← Backup y Restauración                │
├─────────────────────────────────────────┤
│                                         │
│  EXPORTAR                               │
│  ┌─────────────────────────────────┐    │
│  │  📦 Exportar contraseñas        │    │
│  │     Formato: JSON cifrado       │    │
│  │     [EXPORTAR]                  │    │
│  └─────────────────────────────────┘    │
│                                         │
│  IMPORTAR                               │
│  ┌─────────────────────────────────┐    │
│  │  📥 Importar desde CSV          │
│  │     Formato: title,user,pass    │
│  │     [SELECCIONAR ARCHIVO]       │
│  └─────────────────────────────────┘    │
│                                         │
│  ÚLTIMO BACKUP                          │
│  ┌─────────────────────────────────┐    │
│  │  📅 2026-06-10 14:30            │
│  │     45 contraseñas              │
│  └─────────────────────────────────┘    │
│                                         │
└─────────────────────────────────────────┘
```

## Estado

```kotlin
data class BackupState(
    val isLoading: Boolean = false,
    val isExporting: Boolean = false,
    val isImporting: Boolean = false,
    val progress: Float = 0f,
    val successMessage: String? = null,
    val error: String? = null,
    val exportPath: String? = null,
    val importPath: String? = null,
    val lastBackupDate: Long? = null,
    val lastBackupCount: Int = 0
)
```

## Formato de Exportación (JSON)

```json
{
  "version": "1.0",
  "exportDate": "2026-06-10T14:30:00Z",
  "entries": [
    {
      "id": "uuid-123",
      "title": "Google",
      "username": "user@gmail.com",
      "encryptedPassword": "base64-encoded-ciphertext",
      "url": "https://google.com",
      "categoryId": "cat-456",
      "isFavorite": true,
      "createdAt": 1717977000000,
      "updatedAt": 1717977000000
    }
  ],
  "categories": [
    {
      "id": "cat-456",
      "name": "Redes Sociales",
      "color": 4281445632,
      "icon": "FaUsers"
    }
  ]
}
```

## Formato de Importación (CSV)

```csv
title,username,password,url,notes
Google,user@gmail.com,SecurePass123,https://google.com,Cuenta personal
Facebook,user@fb.com,AnotherPass,https://facebook.com,
```

## Use Cases

### ExportEncryptedBackup

```kotlin
class ExportEncryptedBackup(
    private val passwordRepository: PasswordRepository,
    private val categoryRepository: CategoryRepository,
    private val dataCipher: DataCipher,
    private val authRepository: AuthRepository,
    private val context: Context
) {
    suspend operator fun invoke(outputPath: String): Result<Unit> {
        // 1. Verificar sesión activa
        val masterKey = authRepository.getMasterKey()
            ?: return Result.failure(SecurityException("No hay sesión"))
        
        // 2. Obtener todos los datos
        val entries = passwordRepository.getAllEntriesList()
        val categories = categoryRepository.getAllCategories().first()
        
        // 3. Cifrar entradas con clave derivada
        val backupKey = deriveBackupKey(masterKey)
        val encryptedEntries = entries.map { encryptEntry(it, backupKey) }
        
        // 4. Serializar y guardar
        val backupData = BackupData(encryptedEntries, categories)
        val json = Json.encodeToString(backupData)
        File(outputPath).writeText(json)
        
        return Result.success(Unit)
    }
}
```

### ImportFromCSV

```kotlin
class ImportFromCSV(
    private val passwordRepository: PasswordRepository,
    private val context: Context
) {
    suspend operator fun invoke(inputPath: String): Result<Int> {
        val file = File(inputPath)
        val lines = file.readLines()
        var importedCount = 0
        
        for ((index, line) in lines.withIndex()) {
            if (index == 0) continue  // Skip header
            
            val columns = line.split(",")
            if (columns.size >= 3) {
                val entry = PasswordEntry(
                    id = UUID.randomUUID().toString(),
                    title = columns[0].trim(),
                    username = columns[1].trim(),
                    password = columns[2].trim(),
                    url = columns.getOrNull(3)?.trim(),
                    notes = columns.getOrNull(4)?.trim(),
                    categoryId = getDefaultCategoryId(),
                    icon = null,
                    isFavorite = false,
                    createdAt = System.currentTimeMillis(),
                    updatedAt = System.currentTimeMillis()
                )
                
                passwordRepository.insertEntry(entry)
                importedCount++
            }
        }
        
        return Result.success(importedCount)
    }
}
```

## Referencias

- [BackupViewModel](../viewmodels/overview.md#backupviewmodel)
- [ExportEncryptedBackup](../domain/overview.md#exportencryptedbackup)
- [ImportFromCSV](../domain/overview.md#importfromcsv)
