# Pantallas (Screens)

Documentación completa de todas las pantallas de la aplicación.

---

## LoginScreen

Pantalla de autenticación inicial.

**Archivo**: `LoginScreen.kt`

**Estado**:
```kotlin
data class LoginState(
    val password: String = "",
    val isLoading: Boolean = false,
    val error: String? = null,
    val isBiometricAvailable: Boolean = false
)
```

**Eventos**:
- `onPasswordChange(String)` - Usuario escribe password
- `onLoginClick()` - Iniciar sesión
- `onBiometricClick()` - Autenticar con biometría

**Flujo**:
```
Usuario ingresa password → onPasswordChange → Actualiza state
Usuario pulsa "Entrar" → onLoginClick → ViewModel.authenticate()
                                            ↓
                                    Éxito → Navegar a Home
                                    Error → Mostrar error
```

---

## HomeScreen

Pantalla principal de inicio con menú de navegación.

**Archivo**: `HomeScreen.kt`

**Elementos**:
- Header con título de la app
- Grid de opciones de navegación
- Barra de estado con información de sesión

**Opciones de navegación**:

| Opción | Icono | Destino |
|--------|-------|---------|
| Contraseñas | FaLock | PasswordListScreen |
| Generador | FaKey | PasswordGeneratorScreen |
| Categorías | FaFolders | CategoryManagementScreen |
| Copia de Seguridad | FaDownload | BackupScreen |
| Auditoría | FaShield | AuditScreen |
| Estadísticas | FaChartBar | StatisticsScreen |
| Configuración | FaCog | SettingsScreen |

---

## PasswordListScreen

Lista de contraseñas con búsqueda y filtrado.

**Archivo**: `PasswordListScreen.kt`

**Diseño**:
- SearchBar en la parte superior
- Filtro de categorías (chips horizontales)
- Lista dividida en dos secciones:
  - **Favoritos**: Contraseñas marcadas como favoritas
  - **Otras contraseñas**: Resto de contraseñas

**Estado**:
```kotlin
data class PasswordListState(
    val entries: List<PasswordEntry> = emptyList(),
    val searchQuery: String = "",
    val selectedCategoryId: String? = null,
    val isLoading: Boolean = true,
    val error: String? = null
)
```

**Características**:
- Búsqueda en tiempo real (título, username, URL)
- Filtrado por categoría
- Sección de favoritos primero
- Long press para eliminar
- Botón favorito en cada entrada

---

## PasswordDetailScreen

Muestra el detalle de una contraseña específica.

**Archivo**: `PasswordDetailScreen.kt`

**Información mostrida**:
- Título
- Username (con botón de copiar)
- Password (oculta/visible, con botón de copiar)
- URL (clicable)
- Notas
- Categoría (con chip de color)
- Fecha de creación/actualización

**Acciones**:
- Editar → Navega a PasswordFormScreen
- Eliminar → Muestra diálogo de confirmación
- Copiar username → Copia al portapapeles
- Copiar password → Copia al portapapeles

---

## PasswordFormScreen

Formulario para crear o editar contraseñas.

**Archivo**: `PasswordFormScreen.kt`

**Campos**:

| Campo | Tipo | Requerido |
|-------|------|-----------|
| Título | TextField | Sí |
| Username | TextField | Sí |
| Password | TextField + Generator | Sí |
| URL | TextField | No |
| Notas | TextField | No |
| Categoría | Selector | Sí |
| Favorito | Checkbox | No |

**Validaciones**:
- Título no vacío
- Username no vacío
- Password no vacío
- URL válida (si se proporciona)

**Generador integrado**:
- Botón "Generar" abre widget de generación
- Opciones: longitud, tipos de caracteres
- Vista previa en tiempo real

---

## PasswordGeneratorScreen

Generador de contraseñas seguras.

**Archivo**: `PasswordGeneratorScreen.kt`

**Opciones**:
- Longitud (slider 8-128)
- Incluir mayúsculas (checkbox)
- Incluir minúsculas (checkbox)
- Incluir números (checkbox)
- Incluir símbolos (checkbox)

**Características**:
- Vista previa en tiempo real
- Botón de copiar al portapapeles
- Medidor de fortaleza (débil/medio/fuerte)
- Generación con un toque

---

## CategoryManagementScreen

Gestión de categorías personalizadas.

**Archivo**: `CategoryManagementScreen.kt`

**Funcionalidades**:
- Lista de categorías (scroll vertical)
- Botón "Nueva categoría"
- Editar categoría existente (nombre, color, icono)
- Eliminar categoría (solo personalizadas)

**Diálogo de creación/edición**:
- Nombre (TextField)
- Selector de color (paleta predefinida)
- Selector de icono (Font Awesome)

---

## SettingsScreen

Configuración de la aplicación.

**Archivo**: `SettingsScreen.kt`

**Opciones**:

**Apariencia**:
- Tema (Claro/Oscuro/Automático)

**Seguridad**:
- Autenticación biométrica (toggle)
- Timeout de bloqueo automático (selector)
- Cambiar contraseña maestra

**Datos**:
- Exportar backup
- Importar desde CSV
- Borrar todos los datos

---

## BackupScreen

Gestión de copias de seguridad.

**Archivo**: `BackupScreen.kt`

**Exportar**:
- Seleccionar ubicación de archivo
- Cifrado con clave maestra
- Formato: JSON cifrado

**Importar**:
- Seleccionar archivo de backup
- Validar integridad
- Fusionar o reemplazar datos

---

## AuditScreen

Auditoría de seguridad de contraseñas.

**Archivo**: `AuditScreen.kt`

**Métricas**:
- Total de contraseñas
- Contraseñas débiles
- Contraseñas reutilizadas
- Fortaleza promedio

**Lista de problemas**:
- Contraseñas < 8 caracteres
- Sin mayúsculas/números/símbolos
- Patrones comunes detectados

---

## StatisticsScreen

Estadísticas de uso y seguridad.

**Archivo**: `StatisticsScreen.kt`

**Estadísticas**:
- Gráfico de contraseñas por categoría
- Evolución temporal (si hay histórico)
- Porcentaje de fortaleza
- Contraseñas por tipo de servicio

---

## OnboardingScreen

Pantalla de bienvenida para primer uso.

**Archivo**: `OnboardingScreen.kt`

**Contenido**:
- Bienvenida a la aplicación
- Explicación de características
- Configuración de contraseña maestra
- Configuración de biometría (opcional)

---

## ChangePasswordScreen

Cambio de contraseña maestra.

**Archivo**: `ChangePasswordScreen.kt`

**Campos**:
- Contraseña actual
- Nueva contraseña
- Confirmar nueva contraseña

**Proceso**:
1. Verificar contraseña actual
2. Validar nueva contraseña (fortaleza)
3. Confirmar coincidencia
4. Re-cifrar todas las contraseñas con nueva clave
