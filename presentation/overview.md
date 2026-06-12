# Capa de Presentación

## Visión General

La capa de presentación es responsable de:
- Mostrar la interfaz de usuario con Jetpack Compose
- Capturar interacciones del usuario
- Gestionar estados de UI
- Navegación entre pantallas
- Validación de entrada de usuario

## Arquitectura MVVM

```
┌─────────────────────────────────────────────────────────────────┐
│                         VIEW (UI)                                │
│  @Composable screens que observan StateFlow<State>               │
│  Emite eventos → ViewModel                                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       VIEWMODEL                                  │
│  Expone StateFlow<State>                                         │
│  Procesa eventos de UI                                           │
│  Coordina con Use Cases                                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       USE CASES                                  │
│  Lógica de negocio específica                                    │
└─────────────────────────────────────────────────────────────────┘
```

## Estructura de Directorios

```
presentation/
├── ui/
│   ├── screens/                # Pantallas completas
│   │   ├── LoginScreen.kt
│   │   ├── HomeScreen.kt
│   │   ├── PasswordListScreen.kt
│   │   ├── PasswordDetailScreen.kt
│   │   ├── PasswordFormScreen.kt
│   │   ├── PasswordGeneratorScreen.kt
│   │   ├── CategoryManagementScreen.kt
│   │   ├── SettingsScreen.kt
│   │   ├── BackupScreen.kt
│   │   ├── AuditScreen.kt
│   │   ├── StatisticsScreen.kt
│   │   ├── OnboardingScreen.kt
│   │   └── ChangePasswordScreen.kt
│   │
│   ├── components/             # Componentes reutilizables
│   │   ├── SearchBar.kt
│   │   ├── CategoryFilterChips.kt
│   │   ├── CategoryPicker.kt
│   │   ├── PasswordGeneratorWidget.kt
│   │   └── ColorPickerDialog.kt
│   │
│   └── state/                  # Estados de UI
│       ├── PasswordListState.kt
│       ├── PasswordFormState.kt
│       └── PasswordDetailState.kt
│
├── viewmodel/                  # ViewModels
│   ├── AuthViewModel.kt
│   ├── PasswordListViewModel.kt
│   ├── PasswordFormViewModel.kt
│   ├── PasswordDetailViewModel.kt
│   ├── PasswordGeneratorViewModel.kt
│   ├── CategoryManagementViewModel.kt
│   ├── SettingsViewModel.kt
│   ├── BackupViewModel.kt
│   ├── AuditViewModel.kt
│   ├── StatisticsViewModel.kt
│   └── ChangePasswordViewModel.kt
│
├── navigation/
│   └── NavGraph.kt             # Grafo de navegación
│
├── theme/                      # Tema Material 3
│   ├── Color.kt
│   ├── Theme.kt
│   └── Type.kt
│
├── widget/                     # Widgets
│   └── PasswordGeneratorWidget.kt
│
├── MainActivity.kt             # Actividad principal
└── AutoLockManager.kt          # Gestión de bloqueo automático
```

---

## Archivos de Documentación

| Archivo | Contenido |
|---------|-----------|
| [`screens.md`](screens.md) | Las 12 pantallas: Login, Home, PasswordList, PasswordDetail, PasswordForm, PasswordGenerator, CategoryManagement, Settings, Backup, Audit, Statistics, Onboarding, ChangePassword |
| [`components.md`](components.md) | Componentes reutilizables: SearchBar, CategoryFilterChips, CategoryPicker, CategoryDropdown, ColorPickerDialog, PasswordGeneratorWidget |
| [`viewmodels.md`](viewmodels.md) | Los 10 ViewModels con sus estados, eventos y métodos públicos |
| [`theme.md`](theme.md) | Tema Material 3: colores, esquemas, tipografía |
| [`navigation.md`](navigation.md) | Grafo de navegación, rutas, patrones de navegación |

---

## Resumen de Componentes

### Pantallas (12)
- **LoginScreen**: Autenticación inicial con password y biometría
- **HomeScreen**: Menú principal de navegación
- **PasswordListScreen**: Lista con búsqueda y filtrado por categoría
- **PasswordDetailScreen**: Detalle de contraseña individual
- **PasswordFormScreen**: Formulario de creación/edición
- **PasswordGeneratorScreen**: Generador de contraseñas seguras
- **CategoryManagementScreen**: Gestión de categorías personalizadas
- **SettingsScreen**: Configuración de la aplicación
- **BackupScreen**: Exportar/importar copias de seguridad
- **AuditScreen**: Auditoría de contraseñas débiles
- **StatisticsScreen**: Estadísticas de seguridad
- **OnboardingScreen**: Bienvenida y configuración inicial
- **ChangePasswordScreen**: Cambio de contraseña maestra

### Componentes (5)
- **SearchBar**: Búsqueda con icono y botón de limpiar
- **CategoryFilterChips**: Chips horizontales para filtrado
- **CategoryPicker/CategoryDropdown**: Selectores de categoría
- **ColorPickerDialog**: Selector de color para categorías
- **PasswordGeneratorWidget**: Widget de generación en formularios

### ViewModels (10)
Todos siguen el patrón `StateFlow<State>` + eventos de UI:
- `AuthViewModel`, `PasswordListViewModel`, `PasswordFormViewModel`
- `PasswordDetailViewModel`, `PasswordGeneratorViewModel`
- `SettingsViewModel`, `AuditViewModel`, `StatisticsViewModel`
- `BackupViewModel`, `ChangePasswordViewModel`

### Tema
- **Material 3** con soporte para tema dinámico (Android 12+)
- Modo oscuro/claro automático
- Tipografía completa (15 estilos)
