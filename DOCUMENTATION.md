# 📖 Documentación General - Proyecto Albergue

Guía completa de conceptos, funcionalidades y explicaciones del sistema Proyecto Albergue.

## 📋 Tabla de Contenidos

1. [Introducción](#introducción)
2. [Conceptos Fundamentales](#conceptos-fundamentales)
3. [Flujo de la Aplicación](#flujo-de-la-aplicación)
4. [Componentes Principales](#componentes-principales)
5. [Funcionalidades Detalladas](#funcionalidades-detalladas)
6. [Flujo de Datos](#flujo-de-datos)
7. [Estados y Ciclos de Vida](#estados-y-ciclos-de-vida)
8. [Guía de Usuario](#guía-de-usuario)

## 🎯 Introducción

El Proyecto Albergue es un sistema de logística en tiempo real que facilita el seguimiento de entregas de paquetes entre puntos de distribución (albergues) y repartidores. Utiliza MQTT como protocolo de comunicación principal para garantizar eficiencia y baja latencia.

### Problema Que Resuelve

- **Falta de visibilidad**: Los gestores de albergues no sabían cuándo llegaría el repartidor
- **Ineficiencia logística**: Sin rutas optimizadas, los repartidores tardaban más
- **Comunicación lenta**: Métodos antiguos de notificación eran no-reales tiempo

### Solución Implementada

- Sistema de tracking en tiempo real
- Cálculo automático de rutas y ETA
- Comunicación bidireccional instantánea
- Interfaz visual intuitiva con mapas

## 💡 Conceptos Fundamentales

### 1. MQTT (Message Queuing Telemetry Transport)

MQTT es un protocolo de comunicación ligero basado en pub/sub:

```
┌─────────────────────────────────────────┐
│         Broker MQTT                      │
│  (Mosquitto en 192.168.1.137:9001)      │
└─────────────────────────────────────────┘
         ↑           ↑           ↑
         │           │           │
   [Publica]   [Publica]   [Suscribe]
         │           │           │
┌────────┴┐  ┌───────┴──┐  ┌────┴────────┐
│ Repartidor  │ Repartidor  │ Albergue   │
│  (App)     │  (App)     │  (App)      │
└────────────┘ └────────────┘ └────────────┘
```

**Ventajas:**
- Baja latencia
- Bajo uso de ancho de banda
- Escalable
- Confiable con QoS

### 2. Geolocalización

El sistema utiliza GPS/red para obtener:
- Latitud y Longitud
- Precisión
- Altitud

### 3. Mapas Interactivos (Leaflet.js)

Visualización HTML5 de ubicaciones con:
- Tiles de OpenStreetMap
- Markers personalizados
- Polylines para rutas
- Pan y zoom

### 4. ETA (Estimated Time of Arrival)

Tiempo estimado de llegada calculado mediante:
- Distancia euclidiana entre puntos
- Velocidad promedio (≈50 km/h)
- Actualización dinámica

Fórmula básica:
```
Distancia (km) = √[(Δlat)² + (Δlng)²] × 111
ETA (minutos) = Distancia / velocidad × 60
```

## 🔄 Flujo de la Aplicación

### Inicio de Sesión (Albergue)

```
┌─────────────────────────────────────────────────┐
│  1. APP INICIA                                  │
│     - Se inicializa React Native                │
│     - Se cargan componentes                     │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  2. MENÚ PRINCIPAL VISIBLE                      │
│     - Botones de albergues predefinidos         │
│     - Opción de "Mi Ubicación Actual"          │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  3. USUARIO SELECCIONA ALBERGUE                 │
│     - Se obtiene ubicación del albergue         │
│     - Se activa sesión de monitoreo            │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  4. CONEXIÓN MQTT                               │
│     - Se conecta al broker                      │
│     - Se suscribe a 'paquetes/posicion'        │
│     - Espera ubicación del repartidor          │
└─────────────────────────────────────────────────┘
```

### Flujo de Repartidor

```
┌─────────────────────────────────────────────────┐
│  1. APP INICIA                                  │
│     - Solicita permiso de localización          │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  2. INGRESA NOMBRE Y INICIA VIAJE              │
│     - Se guarda nombre del repartidor           │
│     - Conexión a MQTT                          │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  3. COMPARTICIÓN DE UBICACIÓN                   │
│     - Publica ubicación cada 5 segundos         │
│     - Tema: 'paquetes/posicion'                │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  4. RECIBE RUTA                                │
│     - Se suscribe a 'paquetes/ruta'           │
│     - Recibe ruta y destino                    │
│     - Se visualiza en mapa                     │
└─────────────────────────────────────────────────┘
```

## 🧩 Componentes Principales

### App-Albergue

```javascript
App
├── Estados
│   ├── coords (ubicación seleccionada)
│   ├── albergueSeleccionado (datos del albergue)
│   ├── repartidorCoords (ubicación en tiempo real)
│   ├── eta (tiempo estimado)
│   ├── ruta (polyline points)
│   ├── mqttClient (cliente MQTT)
│   └── viajeActivo (sesión activa)
│
├── Efectos (useEffect)
│   ├── Conexión MQTT
│   ├── Actualización de marcadores
│   ├── Visualización de rutas
│   └── Inyección de scripts en WebView
│
├── Funciones
│   ├── obtenerUbicacionActual()
│   ├── seleccionarAlbergue()
│   ├── calcularETAYRuta()
│   └── refWebView.injectJavaScript()
│
└── Vistas
    ├── Menú Principal (selección de albergue)
    └── Mapa + Modal Inferior (monitoreo)
```

### App-Repartidor

```javascript
App
├── Estados
│   ├── coords (ubicación actual)
│   ├── ruta (puntos de ruta)
│   ├── destino (albergue destino)
│   ├── eta (tiempo estimado)
│   ├── nombre (nombre del repartidor)
│   ├── mqttClient (cliente MQTT)
│   └── viajeActivo (sesión activa)
│
├── Efectos (useEffect)
│   ├── Solicitud de permisos
│   ├── Conexión MQTT
│   ├── Publicación de ubicación (intervalo)
│   ├── Suscripción a ruta
│   └── Actualización de mapa
│
├── Funciones
│   ├── solicitarPermiso()
│   ├── iniciarViaje()
│   ├── actualizarUbicacion()
│   ├── mostrarRutaEnMapa()
│   └── terminarViaje()
│
└── Vistas
    ├── Menú Inicio (nombre + botón)
    └── Mapa + Modal Inferior (navegación)
```

## 📊 Funcionalidades Detalladas

### 1. Geolocalización

**Permiso y Acceso:**
- Solicita permiso a nivel de SO
- Obtiene ubicación actual o exacta
- Proporciona precisión de ±5-10 metros

**Código Relevante:**
```javascript
const obtenerUbicacionActual = async () => {
  const { status } = await Location.requestForegroundPermissionsAsync();
  if (status !== 'granted') {
    Alert.alert('Permiso denegado', 'Se necesita permiso de ubicación');
    return;
  }
  
  try {
    const location = await Location.getCurrentPositionAsync({});
    setCoords(location.coords);
  } catch (err) {
    console.error('Error:', err);
  }
};
```

### 2. Conexión MQTT

**Configuración:**
- Protocolo: WebSocket Seguro (ws://)
- Host: 192.168.1.137
- Puerto: 9001
- Ruta: /mqtt

**Ciclo de Vida:**
1. Crear cliente con ID único
2. Conectar al broker
3. Suscribirse a temas necesarios
4. Publicar mensajes
5. Desconectar al terminar

### 3. Cálculo de Ruta

**Algoritmo:**
```
1. Obtener ubicación origen (repartidor)
2. Obtener ubicación destino (albergue)
3. Calcular distancia euclidiana
4. Interpolar puntos intermedios
5. Crear polyline suave
6. Enviar al repartidor
```

**Interpolación:**
```javascript
const rutaCoords = [];
const numPuntos = Math.min(50, Math.max(10, Math.floor(distancia / 0.5)));

for (let i = 0; i <= numPuntos; i++) {
  const t = i / numPuntos;
  const lat = origen.lat + (destino.lat - origen.lat) * t;
  const lng = origen.lng + (destino.lng - origen.lng) * t;
  rutaCoords.push([lat, lng]);
}
```

### 4. Visualización en Mapa

**Elementos Mapa:**

| Elemento | Ícono | Color | Rota |
|----------|-------|-------|------|
| Repartidor | 📍 | Rojo | Sí |
| Albergue | 📦 | Azul | No |
| Ruta | — | Verde | — |

**Zooming Automático:**
- Al recibir ubicación del repartidor
- Al mostrar ruta (fitBounds)
- Centrado en albergue al iniciar

### 5. Comunicación en Tiempo Real

**Temas MQTT:**

| Tema | Origen | Destino | Frecuencia |
|------|--------|---------|-----------|
| paquetes/posicion | Repartidor | Albergue | Cada 5s |
| paquetes/ruta | Albergue | Repartidor | Cuando se calcula |

## 🔄 Flujo de Datos

```
┌──────────────────────────────────────────────────────────────┐
│                      FLUJO DE DATOS                          │
└──────────────────────────────────────────────────────────────┘

         ┌─────────────────────────────────────┐
         │   Repartidor (App)                  │
         │  - Obtiene ubicación GPS            │
         │  - Cada 5 segundos                  │
         └─────────────────────────────────────┘
                         ↓ (Publica)
         ┌─────────────────────────────────────┐
         │   Broker MQTT                       │
         │   Tema: paquetes/posicion           │
         │   {                                 │
         │     latitude: 20.375,               │
         │     longitude: -99.982,             │
         │     timestamp: 1234567890           │
         │   }                                 │
         └─────────────────────────────────────┘
                         ↓ (Suscrito)
         ┌─────────────────────────────────────┐
         │   Albergue (App)                    │
         │  - Recibe ubicación                 │
         │  - Calcula distancia                │
         │  - Calcula ETA                      │
         │  - Genera ruta                      │
         └─────────────────────────────────────┘
                         ↓ (Publica)
         ┌─────────────────────────────────────┐
         │   Broker MQTT                       │
         │   Tema: paquetes/ruta               │
         │   {                                 │
         │     ruta: [[lat,lng], ...],         │
         │     destino: {lat, lng},            │
         │     eta: 15                         │
         │   }                                 │
         └─────────────────────────────────────┘
                         ↓ (Suscrito)
         ┌─────────────────────────────────────┐
         │   Repartidor (App)                  │
         │  - Recibe ruta                      │
         │  - Recibe destino                   │
         │  - Visualiza en mapa                │
         │  - Navega a destino                 │
         └─────────────────────────────────────┘

```

## 🔀 Estados y Ciclos de Vida

### Estados de la App-Albergue

```
┌─────────────────────┐
│   INICIO            │
│ (Menú Principal)    │
└──────────┬──────────┘
           │
    [Usuario selecciona albergue]
           │
           ↓
┌─────────────────────┐
│   CONECTANDO MQTT   │
│                     │
└──────────┬──────────┘
           │
    [Se conecta al broker]
           │
           ↓
┌─────────────────────┐
│   ESPERANDO         │
│ REPARTIDOR          │
│ (Modal inferior)    │
└──────────┬──────────┘
           │
    [Recibe ubicación del repartidor]
           │
           ↓
┌─────────────────────┐
│   MONITOREO ACTIVO  │
│ - Actualiza posición│
│ - Calcula ETA       │
│ - Envía ruta        │
└──────────┬──────────┘
           │
    [Usuario presiona cerrar]
           │
           ↓
┌─────────────────────┐
│   DESCONECTANDO     │
│ (Vuelve al menú)    │
└─────────────────────┘
```

### Ciclo de Vida MQTT

```
create() → configure() → connect() → subscribe() 
                    ↓
            [onMessageArrived]
                    ↓
            [procesamiento] → publish()
                    ↓
            disconnect() → destroy()
```

## 👥 Guía de Usuario

### Para el Gestor del Albergue

#### Paso 1: Inicio
1. Abre la app "App Albergue"
2. Verás un menú con 3 albergues predefinidos
3. Presiona el botón de tu albergue o "Mi Ubicación Actual"

#### Paso 2: Monitoreo
1. Se abre el mapa automáticamente
2. Aparece un modal en la parte inferior
3. Estado MQTT se muestra en verde (Conectado)
4. **Espera a que llegue el repartidor**

#### Paso 3: Seguimiento
1. Cuando el repartidor sale, verás su ubicación en el mapa (marcador 📍)
2. El modal actualiza su posición y ETA
3. La ruta se visualiza en el mapa (línea punteada azul)
4. El ETA se actualiza dinámicamente

#### Paso 4: Finalización
1. Presiona el botón "✕ Cerrar" en el modal
2. Vuelves al menú principal
3. La conexión MQTT se cierra

### Para el Repartidor

#### Paso 1: Inicio
1. Abre la app "App Repartidor"
2. Ingresa tu nombre (ej: "Juan Pérez")
3. Presiona "🚀 Iniciar Viaje"

#### Paso 2: Permiso de Localización
1. Se solicita permiso para acceder a tu ubicación
2. Presiona "Permitir" o "Aceptar"
3. La app comienza a enviar tu ubicación cada 5 segundos

#### Paso 3: Navegación
1. En el mapa verás tu ubicación (📍)
2. Cuando el albergue recibe tu información, envía la ruta
3. Verás una línea verde con el camino a seguir
4. El albergue aparece como 📦 en azul

#### Paso 4: Finalización
1. Cuando llegues al albergue, presiona "✕ Cerrar"
2. Se detiene el envío de ubicación
3. Vuelves al menú de inicio

---

**Última actualización**: Noviembre 2025
**Versión**: 1.0.0
