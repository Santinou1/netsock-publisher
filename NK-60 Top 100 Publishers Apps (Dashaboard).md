
## Introducción

El dashboard que realize es una plataforma de análisis de datos para aplicaciones móviles que permite a los usuarios explorar y filtrar información sobre publishers (desarrolladores) y sus aplicaciones. El dashboard principal proporciona una visión general de los publishers más populares en diferentes categorías (free, paid, rated) y permite aplicar diversos filtros para analizar datos específicos.

## Estructura del Proyecto

El proyecto sigue una arquitectura basada en Vue 3 con Composition API y está organizado de la siguiente manera:

```

src/

├── api/

│   └── services/

│       └── mobsi/

│           ├── Publishers/

│           │   ├── hooks.ts

│           │   └── index.ts

│           └── TopApps/

├── composables/

│   ├── useAppTags.ts

│   ├── useCountryFilters.ts

│   ├── useDashboard.ts

│   ├── useGeographyFilters.ts

│   ├── usePublisherDetails.ts

│   ├── usePublisherHQFilters.ts

│   └── useTopApps.ts

├── pages/

│   └── netsocks/

│       └── mobsi/

│           ├── dashboard.vue

│           └── publisher/

│               └── [id].vue

└── constants/

    └── filters.ts

```


## Componentes Principales

### Dashboard (`dashboard.vue`)

El dashboard es la página principal de la aplicación y muestra:

1. **Panel de Filtros**: Permite a los usuarios filtrar publishers por:

   - HQ del Editor (país de origen)
   - Etiquetas (categorías)
   - Tienda (App Store, Google Play)

2. **Filtro de Geografía**: Permite ver las apps populares por país específico.

3. **Rankings de Publishers**:
   - Top Free: Publishers con más descargas gratuitas
   - Top Paid: Publishers con más descargas pagas
   - Top Rated: Publishers con mejor calificación

4. **Top 100 Apps**: Cuando se selecciona un país en el filtro de geografía, muestra:
   - Top Free Apps
   - Top Paid Apps
   - Top Grossing Apps


### Página de Detalles del Publisher (`publisher/[id].vue`)

Esta página muestra información detallada sobre un publisher específico:

1. **Información General**: Nombre, país, estadísticas generales.

2. **Métricas Clave**:
   - Total de Apps
   - Descargas Totales
   - Rating Promedio
   - Ingresos Estimados

3. **Lista de Apps**: Todas las aplicaciones del publisher con detalles como:

   - Título
   - Categoría
   - Descargas
   - Rating
   - Fecha de creación y actualización


## Servicios y API

### Publishers Service (`Publishers/index.ts`)
  
Este servicio gestiona las llamadas a la API relacionadas con publishers:

1. **Endpoints Normales**:

   - `getTopPublishers(country?, category?)`: Obtiene los publishers más populares con filtros opcionales
   - `getTopPaidPublishers(country?, category?)`: Obtiene los publishers con más apps pagas
   - `getTopRatedPublishers(country?, category?)`: Obtiene los publishers mejor calificados

  

2. **Endpoints Cacheados** (Estos son los que comente anteriormente con las nuevas colecciones):

   - `getCachedTopFreePublishers()`: Obtiene los publishers más populares desde un endpoint cacheado
   - `getCachedTopPaidPublishers()`: Obtiene los publishers con más apps pagas desde un endpoint cacheado
   - `getCachedTopGrossingPublishers()`: Obtiene los publishers con más ingresos desde un endpoint cacheado

Aca me encontré con una problemática que no tuve tiempo de resolver, y no queria hacer "trampa" y resolverlo en el finde largo debido que mi tiempo de prueba era hasta el viernes, y no se si se iba a ver de buena manera. 
El problema es que, al filtrar por país o categorías, la aplicación se demora debido a que realiza la solicitud a los endpoints antiguos, es decir, a los que primero hacen el _aggregate_ y luego filtran los resultados.

No utilicé los filtros con los nuevos endpoints porque solo devuelven 100 resultados. Si aplicaba un filtro, estaría limitando esos 100 resultados en lugar de filtrar sobre todas las aplicaciones. Por ejemplo, si filtraba usando los endpoints cacheados con el parámetro "country": "us", de esos 100 resultados solo obtenía 40, ya que solo se consideraban esos 100 que estaban en el front.

Lo que quería lograr era obtener el top 100 de todas las aplicaciones que corresponden al país "US", y lo mismo con las categorías.

### Hooks de Publishers (`Publishers/hooks.ts`)

Estos hooks utilizan vue Query para gestionar el estado, la caché y las solicitudes a la API:
  
1. **useTopPublishers**: Gestiona la obtención de los publishers más populares
2. **useTopPaidPublishers**: Gestiona la obtención de los publishers con más apps pagas
3. **useTopRatedPublishers**: Gestiona la obtención de los publishers mejor calificados

  

Cada hook implementa lógica para:
- Usar endpoints cacheados en la carga inicial
- Usar endpoints normales cuando se aplican filtros
- Volver a los endpoints cacheados cuando se restablecen los filtros


## Composables
### `useDashboard.ts`

Composable central que gestiona el estado del dashboard:
  
- **Estado de filtros**: país, categoría, tienda, etc.
- **Queries de datos**: Utiliza los hooks de publishers para obtener datos

- **Métodos**:
  - `updateFilters`: Actualiza los filtros y recarga los datos
  - `clearFilters`: Restablece los filtros y vuelve a los datos cacheados
- **Datos computados**: Transforma los datos de la API en rankings para mostrar en la UI


### `useAppTags.ts`

Gestiona las etiquetas (categorías) de aplicaciones:  
- Lista de etiquetas populares
- Búsqueda y filtrado de etiquetas
- Selección y deselección de etiquetas

### `useCountryFilters.ts` y `usePublisherHQFilters.ts`

Gestionan los filtros de países:
- Lista de países populares
- Búsqueda y filtrado de países
- Selección y deselección de países

### `useGeographyFilters.ts`
Gestiona el filtro de geografía para ver apps populares por país:

- A diferencia de otros filtros, solo permite seleccionar un país a la vez
- Implementa lógica para evitar conflictos con otros filtros

### `useTopApps.ts`
Gestiona la obtención de las top apps por país:
  
- Top Free Apps
- Top Paid Apps
- Top Grossing Apps
### `usePublisherDetails.ts`

Gestiona los datos para la página de detalles del publisher:
- Información general del publisher
- Lista de apps del publisher
- Estadísticas y métricas
## Flujo de Datos

1. **Carga Inicial**:
   - Se cargan los publishers desde endpoints cacheados para mejorar el rendimiento
   - No se aplican filtros inicialmente

2. **Aplicación de Filtros**:
   - El usuario selecciona filtros (país, categoría, etiqueta, etc.)
   - Se llama a `updateFilters` en `useDashboard`
   - Se obtienen datos filtrados desde los endpoints normales

3. **Restablecimiento de Filtros**:
   - El usuario hace clic en "Restablecer filtros"
   - Se llama a `clearFilters` en `useDashboard`
   - Se vuelven a cargar los datos desde los endpoints cacheados

4. **Navegación a Detalles**:
   - El usuario hace clic en un publisher
   - Se navega a la página de detalles con el ID del publisher
   - `usePublisherDetails` carga los datos específicos del publisher

