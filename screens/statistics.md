# StatisticsScreen

## Visión General

Pantalla que muestra estadísticas y métricas sobre las contraseñas guardadas.

**Archivo**: `presentation/ui/screens/StatisticsScreen.kt`

## Estructura de la UI

```
┌─────────────────────────────────────────┐
│  ← Estadísticas                         │
├─────────────────────────────────────────┤
│                                         │
│  RESUMEN                                │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │   45    │ │   12    │ │   8     │   │
│  │ Total   │ │ Favorit │ │ Débiles │   │
│  └─────────┘ └─────────┘ └─────────┘   │
│                                         │
│  POR CATEGORÍA                          │
│  ┌─────────────────────────────────┐    │
│  │ Redes Sociales  ████████ 12     │    │
│  │ Finanzas        ██████ 8        │    │
│  │ Compras         ████ 5          │    │
│  │ Trabajo         ███ 4           │    │
│  │ Otros           ██ 3            │    │
│  └─────────────────────────────────┘    │
│                                         │
│  FORTALEZA                              │
│  ┌─────────────────────────────────┐    │
│  │ Muy Fuerte  ████████ 15 (33%)   │    │
│  │ Fuerte      ██████ 10 (22%)     │    │
│  │ Media       ████ 8 (18%)        │    │
│  │ Débil       ██ 5 (11%)          │    │
│  │ Muy Débil   █ 2 (4%)            │    │
│  └─────────────────────────────────┘    │
│                                         │
│  EDAD DE CONTRASEÑAS                    │
│  ┌─────────────────────────────────┐    │
│  │ < 30 días       10              │    │
│  │ 30-90 días      20              │    │
│  │ > 90 días       15 ⚠️           │    │
│  └─────────────────────────────────┘    │
│                                         │
└─────────────────────────────────────────┘
```

## Estado

```kotlin
data class StatisticsState(
    val totalPasswords: Int = 0,
    val favoriteCount: Int = 0,
    val categoryCount: Int = 0,
    val weakPasswordCount: Int = 0,
    val reusedPasswordCount: Int = 0,
    val oldPasswordCount: Int = 0,
    val averageStrength: Double = 0.0,
    val categoryDistribution: Map<String, Int> = emptyMap(),
    val strengthDistribution: StrengthDistribution = StrengthDistribution(),
    val isLoading: Boolean = false,
    val error: String? = null
)

data class StrengthDistribution(
    val veryWeak: Int = 0,   // 0-20
    val weak: Int = 0,       // 21-40
    val medium: Int = 0,     // 41-60
    val strong: Int = 0,     // 61-80
    val veryStrong: Int = 0  // 81-100
)
```

## Métricas

| Métrica | Descripción |
|---------|-------------|
| Total Passwords | Número total de contraseñas guardadas |
| Favorite Count | Contraseñas marcadas como favoritas |
| Category Count | Número de categorías utilizadas |
| Weak Password Count | Contraseñas con puntuación < 40 |
| Reused Password Count | Contraseñas repetidas (mismo password) |
| Old Password Count | Contraseñas sin cambiar > 90 días |
| Average Strength | Fortaleza promedio (0-100) |

## Referencias

- [StatisticsViewModel](../viewmodels/overview.md#statisticsviewmodel)
- [GetSecurityStatistics Use Case](../domain/overview.md#getsecuritystatistics)
