# CategoryManagementScreen

## Visión General

Pantalla para gestionar categorías: ver, crear, editar y eliminar categorías personalizadas.

**Archivo**: `presentation/ui/screens/CategoryManagementScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  ← Gestión de Categorías     [+]        │
├─────────────────────────────────────────┤
│                                         │
│  PREDEFINIDAS                           │
│  ┌─────────────────────────────────┐    │
│  │ 📁 General               (lock) │    │
│  │ 👥 Redes Sociales        (lock) │    │
│  │ 💰 Finanzas              (lock) │    │
│  │ 🛒 Compras               (lock) │    │
│  └─────────────────────────────────┘    │
│                                         │
│  PERSONALIZADAS                         │
│  ┌─────────────────────────────────┐    │
│  │ 🎮 Juegos                [✏️][🗑️]│    │
│  │ 🏋️ Deportes              [✏️][🗑️]│    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │   Nueva Categoría: [_______]    │    │
│  │   Color: [●] Icono: [📁]        │    │
│  │   [GUARDAR]                     │    │
│  └─────────────────────────────────┘    │
│                                         │
└─────────────────────────────────────────┘
```

## Estado

```kotlin
data class CategoryManagementState(
    val categories: List<Category> = emptyList(),
    val predefinedCategories: List<Category> = emptyList(),
    val customCategories: List<Category> = emptyList(),
    val newCategoryName: String = "",
    val newCategoryColor: Int = 0xFF3498DB.toInt(),
    val newCategoryIcon: String = "FaFolder",
    val isLoading: Boolean = false,
    val error: String? = null,
    val successMessage: String? = null,
    val showColorPicker: Boolean = false,
    val showIconPicker: Boolean = false,
    val categoryToDelete: Category? = null,
    val categoryToEdit: Category? = null
)
```

## Reglas de Negocio

| Acción | Permitido | Razón |
|--------|-----------|-------|
| Ver categorías | ✅ Siempre | Visualización |
| Crear categoría | ✅ Siempre | Usuario puede crear |
| Editar predefinida | ❌ No | `isDeletable = false` |
| Editar personalizada | ✅ Sí | `isDeletable = true` |
| Eliminar predefinida | ❌ No | Protegidas |
| Eliminar personalizada | ✅ Sí | Con migración a "General" |

## Use Cases

### CreateCategory

```kotlin
class CreateCategory(private val repository: CategoryRepository) {
    suspend operator fun invoke(category: Category): Result<Unit> {
        if (category.name.isBlank()) {
            return Result.failure(EmptyCategoryNameException())
        }
        
        val newCategory = category.copy(
            isCustom = true,
            isDeletable = true
        )
        
        repository.createCategory(newCategory)
        return Result.success(Unit)
    }
}
```

### DeleteCategory

```kotlin
class DeleteCategory(
    private val repository: CategoryRepository,
    private val passwordRepository: PasswordRepository
) {
    suspend operator fun invoke(category: Category): Result<Unit> {
        // No permitir eliminar predefinidas
        if (!category.isDeletable) {
            return Result.failure(CannotDeleteSystemCategoryException())
        }
        
        // Mover contraseñas a "General"
        val generalCategory = getGeneralCategory()
        val entries = passwordRepository.getEntriesByCategory(category.id).first()
        
        entries.forEach { entry ->
            passwordRepository.updateEntry(entry.copy(categoryId = generalCategory.id))
        }
        
        repository.deleteCategory(category)
        return Result.success(Unit)
    }
}
```

## Referencias

- [CategoryManagementViewModel](../viewmodels/overview.md#categorymanagementviewmodel)
- [CategoryPicker](../components/category-picker.md)
