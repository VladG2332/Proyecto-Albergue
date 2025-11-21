# 🏗️ Arquitectura del Sistema - Proyecto Albergue

Descripción detallada de la arquitectura, componentes y sus interacciones.

## 📋 Tabla de Contenidos

1. [Arquitectura General](#arquitectura-general)
2. [Componentes del Sistema](#componentes-del-sistema)
3. [Flujo de Datos](#flujo-de-datos)
4. [Patrones de Diseño](#patrones-de-diseño)
5. [Seguridad](#seguridad)
6. [Escalabilidad](#escalabilidad)
7. [Despliegue](#despliegue)

## 🏛️ Arquitectura General

### Vista de Alto Nivel

```
┌─────────────────────────────────────────────────────────────┐
│                     CLIENTE (App Albergue)                  │
│  React Native + WebView + MQTT                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────────────┐         ┌──────────────────┐       │
│   │   React Layer    │         │   WebView Layer  │       │
│   │  (UI Component)  │         │  (HTML/Leaflet)  │       │
│   └────────┬─────────┘         └────────┬─────────┘       │
│            │                            │                 │
│   ┌────────▼────────────────────────────▼────────┐       │
│   │   MQTT Layer                                  │       │
│   │  (Pub/Sub Communication)                     │       │
│   └────────┬────────────────────────────────────┘       │
│            │                                              │
│   ┌────────▼───────────────────────────────────┐       │
│   │   Native APIs                               │       │
│   │  (Location, Permissions, WebView)           │       │
│   └────────────────────────────────────────────┘       │
│                                                         │
└─────────────────────────────────────────────────────────────┘
                         ↕ (WebSocket)
┌─────────────────────────────────────────────────────────────┐
│              SERVIDOR (Broker MQTT)                         │
│  Mosquitto @ 192.168.1.137:9001                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────────────────────────────────────┐           │
│   │   MQTT Broker (Message Queue)            │           │
│   │   Temas:                                 │           │
│   │   - paquetes/posicion                    │           │
│   │   - paquetes/ruta                        │           │
│   └──────────────────────────────────────────┘           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                         ↕ (WebSocket)
┌─────────────────────────────────────────────────────────────┐
│                     CLIENTE (App Repartidor)                │
│  React Native + WebView + MQTT                              │
├─────────────────────────────────────────────────────────────┤
│  (Estructura similar a App Albergue)                        │
└─────────────────────────────────────────────────────────────┘
```

### Capas de la Arquitectura

#### 1. Capa de Presentación (UI)

**Ubicación**: `App.js` - Componentes React Native

**Componentes**:
- `View`: Contenedor
- `Text`: Texto
- `ScrollView`: Lista desplazable
- `TouchableOpacity`: Botones interactivos
- `TextInput`: Entrada de texto
- `WebView`: Renderizador HTML/Leaflet
- `Modal`: Ventanas flotantes

**Responsabilidades**:
- Renderizar interfaz
- Capturar eventos de usuario
- Mostrar datos de estado
- Inyectar scripts en WebView

#### 2. Capa de Lógica de Negocio

**Ubicación**: Hooks (useState, useEffect)

**Responsabilidades**:
- Gestionar estado de la aplicación
- Procesar datos de entrada
- Coordinar efectos secundarios
- Cálculos de ETA y ruta

#### 3. Capa de Comunicación

**Ubicación**: MQTT Client (Paho)

**Responsabilidades**:
- Conexión al broker
- Publicar mensajes
- Suscribirse a temas
- Manejar desconexiones

**Protocolo**:
```
Cliente → WebSocket → Broker MQTT ← WebSocket ← Cliente
           Port 9001
```

#### 4. Capa de Integración Nativa

**Ubicación**: APIs de Expo

**Componentes**:
- `expo-location`: GPS
- `expo-permissions`: Permisos del SO
- `react-native-webview`: Mapas HTML
- `AsyncStorage`: Almacenamiento local

#### 5. Capa de Servicios Externos

**Ubicación**: Servidores externos

**Servicios**:
- Broker MQTT (Mosquitto)
- OpenStreetMap (Tiles de mapas)
- Leaflet CDN (Librerías JS)

## 🧩 Componentes del Sistema

### Componente: App-Albergue

```
┌────────────────────────────────────────────┐
│          App Component (App-Albergue)       │
├────────────────────────────────────────────┤
│                                            │
│  States (useState)                         │
│  ├── coords                                │
│  ├── albergueSeleccionado                 │
│  ├── repartidorCoords                     │
│  ├── repartidorNombre                     │
│  ├── eta                                  │
│  ├── ruta                                 │
│  ├── lastUpdate                           │
│  ├── mqttClient                           │
│  ├── mqttConnected                        │
│  └── viajeActivo                          │
│                                            │
│  Effects (useEffect)                       │
│  ├── MQTT Connection Effect                │
│  ├── Albergue Marker Update Effect        │
│  ├── Repartidor Position Update Effect    │
│  └── Ruta Display Effect                  │
│                                            │
│  Functions                                 │
│  ├── obtenerUbicacionActual()             │
│  ├── seleccionarAlbergue()                │
│  ├── calcularETAYRuta()                   │
│  └── WebView Script Injection              │
│                                            │
│  Views                                     │
│  ├── MenuView (if !viajeActivo)          │
│  │   ├── ScrollView                       │
│  │   ├── AlbergueButtons[]                │
│  │   └── UbicacionButton                  │
│  │                                         │
│  └── MapView (if viajeActivo)            │
│      ├── WebView (Leaflet Map)            │
│      └── ModalOverlay                     │
│          ├── MqttStatus                   │
│          ├── AlbergueInfo                 │
│          ├── RepartidorInfo               │
│          └── EtaDisplay                   │
│                                            │
└────────────────────────────────────────────┘
```

### Componente: App-Repartidor

```
┌────────────────────────────────────────────┐
│        App Component (App-Repartidor)      │
├────────────────────────────────────────────┤
│                                            │
│  Estados (useState)                        │
│  ├── coords                                │
│  ├── ruta                                 │
│  ├── destino                              │
│  ├── eta                                  │
│  ├── nombre                               │
│  ├── mqttClient                           │
│  ├── mqttConnected                        │
│  ├── webViewReady                         │
│  └── viajeActivo                          │
│                                            │
│  Efectos (useEffect)                       │
│  ├── Location Permission Effect            │
│  ├── MQTT Connection Effect                │
│  ├── Location Update Interval Effect       │
│  ├── Ruta Subscription Effect              │
│  └── Map Update Effects                   │
│                                            │
│  Funciones                                 │
│  ├── solicitarPermiso()                   │
│  ├── iniciarViaje()                       │
│  ├── actualizarUbicacion()                │
│  ├── mostrarRutaEnMapa()                  │
│  └── terminarViaje()                      │
│                                            │
│  Vistas                                    │
│  ├── MenuView (if !viajeActivo)          │
│  │   ├── TextInput (Nombre)               │
│  │   └── StartButton                      │
│  │                                         │
│  └── MapView (if viajeActivo)            │
│      ├── WebView (Leaflet Map)            │
│      └── ModalOverlay                     │
│          ├── RepartidorInfo               │
│          ├── DestinoInfo                  │
│          ├── RutaDisplay                  │
│          └── EtaDisplay                   │
│                                            │
└────────────────────────────────────────────┘
```

### Componente: MQTT Broker

```
┌──────────────────────────────────────────┐
│     Mosquitto MQTT Broker                │
│     (192.168.1.137:9001)                 │
├──────────────────────────────────────────┤
│                                          │
│  Topics                                  │
│  └── paquetes/                           │
│      ├── posicion                        │
│      │   └── [subscribers: albergues]    │
│      └── ruta                            │
│          └── [subscribers: repartidores] │
│                                          │
│  Conexiones WebSocket                    │
│  ├── App Albergue #1                     │
│  ├── App Albergue #2                     │
│  ├── App Repartidor #1                   │
│  ├── App Repartidor #2                   │
│  └── ...                                 │
│                                          │
│  Message Queue                           │
│  └── Almacena mensajes con QoS=1         │
│                                          │
└──────────────────────────────────────────┘
```

## 🔄 Flujo de Datos

### Flujo Completo: Desde Repartidor Hasta Albergue

```
1. REPARTIDOR PUBLICA UBICACIÓN
   ┌──────────────────────────────┐
   │ App Repartidor               │
   │ - obtenerUbicacion()         │
   │ - setCoords()                │
   │ - mqttClient.send()          │
   └──────────────────────────────┘
                ↓
   Mensaje:
   {
     "latitude": 20.375,
     "longitude": -99.982,
     "timestamp": 1700000000000
   }

2. ENVÍA A BROKER MQTT
   ┌──────────────────────────────┐
   │ WebSocket                    │
   │ ws://192.168.1.137:9001      │
   │ Tema: paquetes/posicion      │
   └──────────────────────────────┘

3. ALBERGUE RECIBE UBICACIÓN
   ┌──────────────────────────────┐
   │ App Albergue                 │
   │ - onMessageArrived()         │
   │ - setRepartidorCoords()      │
   │ - setLastUpdate()            │
   └──────────────────────────────┘

4. ALBERGUE CALCULA ETA Y RUTA
   ┌──────────────────────────────┐
   │ calcularETAYRuta()           │
   │ - Calcula distancia          │
   │ - Interpola puntos           │
   │ - Calcula minutos            │
   │ - setEta()                   │
   │ - setRuta()                  │
   └──────────────────────────────┘

5. ALBERGUE PUBLICA RUTA
   ┌──────────────────────────────┐
   │ mqttClient.send()            │
   │ Tema: paquetes/ruta          │
   └──────────────────────────────┘
   
   Mensaje:
   {
     "ruta": [[lat,lng], ...],
     "destino": {lat, lng},
     "eta": 15
   }

6. REPARTIDOR RECIBE RUTA
   ┌──────────────────────────────┐
   │ App Repartidor               │
   │ - onMessageArrived()         │
   │ - setRuta()                  │
   │ - setDestino()               │
   │ - setEta()                   │
   └──────────────────────────────┘

7. ACTUALIZA VISUALIZACIÓN
   ┌──────────────────────────────┐
   │ WebView                      │
   │ - mostrarRuta()              │
   │ - mostrarDestino()           │
   │ - Renderiza polyline         │
   │ - Renderiza markers          │
   └──────────────────────────────┘
```

### Diagrama de Secuencia

```
Repartidor        Broker MQTT         Albergue
   │                  │                   │
   │──Ubicación (5s)──>│                   │
   │                  │──Ubicación────>│
   │                  │                   │
   │                  │<──Ruta-----------│
   │<──────Ruta───────│                   │
   │                  │                   │
   │──Ubicación (5s)──>│                   │
   │                  │──Ubicación────>│
   │                  │                   │
   │                  │<──Ruta-----------│
   │<──────Ruta───────│                   │
   │                  │                   │
   [Repartidor navega y llega al albergue]
```

## 🎨 Patrones de Diseño

### 1. Patrón Componente (React)

Cada aplicación es un componente funcional principal que gestiona todo:

```javascript
export default function App() {
  // Lógica centralizada
  return (
    // Renderizado condicional
  );
}
```

**Ventajas**:
- Fácil de entender
- Gestión centralizada de estado
- Rápido prototipado

**Desventajas**:
- Componente muy grande (500+ líneas)
- Difícil de testear
- Reutilización limitada

### 2. Patrón Pub/Sub (MQTT)

Publicadores y suscriptores desacoplados:

```
Repartidor (Publisher) ──┐
                        ├→ Broker MQTT ──┬→ Albergue (Subscriber)
                                        └→ Otros Albergues
```

**Ventajas**:
- Desacoplamiento de componentes
- Escalabilidad (múltiples suscriptores)
- Bajo acoplamiento

### 3. Patrón State Management (Hooks)

React Hooks para manejo de estado:

```javascript
// Estado
const [estado, setEstado] = useState(inicial);

// Efectos secundarios
useEffect(() => {
  // Lógica
}, [dependencias]);
```

**Ventajas**:
- Simple y directo
- Integrado en React
- Reutilizable

### 4. Patrón Inyección de Script (WebView)

Comunicación entre React Native y HTML:

```javascript
// React Native
refWebView.current.injectJavaScript(`
  window.updatePosicion(${JSON.stringify(coords)});
`);

// HTML/JavaScript
window.updatePosicion = function(data) {
  // Actualizar mapa
};
```

## 🔒 Seguridad

### Consideraciones de Seguridad Actuales

#### 1. MQTT Sin Encriptación

**Riesgo**: Bajo (red local)

```
Actual: ws://192.168.1.137:9001 (Sin SSL)
Recomendado: wss://192.168.1.137:8883 (Con SSL)
```

**Recomendación**: Para producción, usar:
```javascript
const BROKER = 'wss://broker.example.com:8883';
client.connect({
  useSSL: true,
  userName: 'user',
  password: 'pass'
});
```

#### 2. Sin Autenticación MQTT

**Riesgo**: Cualquiera puede conectarse

**Solución**:
```javascript
client.connect({
  useSSL: true,
  userName: 'albergue',
  password: 'securePassword123',
  onSuccess: () => { /* ... */ }
});
```

#### 3. IDs de Cliente Públicos

**Riesgo**: Identificación de usuarios

**Actual**:
```javascript
new Client(BROKER_HOST, BROKER_PORT, BROKER_PATH, 'albergue-' + Math.random());
```

**Mejorado**:
```javascript
new Client(BROKER_HOST, BROKER_PORT, BROKER_PATH, 
           'albergue-' + generateSecureToken());
```

#### 4. Validación de Datos

**Actual**: Mínima validación

**Recomendación**:
```javascript
const validarCoordenadas = (coords) => {
  return coords &&
         typeof coords.latitude === 'number' &&
         typeof coords.longitude === 'number' &&
         Math.abs(coords.latitude) <= 90 &&
         Math.abs(coords.longitude) <= 180;
};
```

### Checklist de Seguridad

- [ ] Usar MQTT con SSL/TLS (wss://)
- [ ] Implementar autenticación (user/pass)
- [ ] Validar datos recibidos
- [ ] Implementar autorización (tokens)
- [ ] Usar HTTPS para APIs HTTP
- [ ] Implementar rate limiting
- [ ] Registrar eventos de seguridad
- [ ] Mantener dependencias actualizadas

## 📈 Escalabilidad

### Crecimiento Horizontal

**Problema Actual**: Un broker MQTT centralizado

**Solución - Broker Distribuido**:
```
          ┌─── Broker 1 ───┐
         /                  \
   Cliente 1            Cliente 2
     (App)                (App)
         \                  /
          └─── Broker 2 ───┘
```

### Crecimiento Vertical

**Optimizaciones**:

1. **Compresión de Mensajes**:
```javascript
// Actual: 500+ bytes
const mensaje = JSON.stringify({
  ruta: rutaCoords,
  destino: { lat, lng },
  eta: minutos
});

// Optimizado: ~200 bytes
const mensajeComprimido = {
  r: rutaCoords,
  d: { lat, lng },
  e: minutos
};
```

2. **Throttling de Ubicación**:
```javascript
// Actual: Cada 5 segundos
// Optimizado: Cada 10 segundos o cambio > 50m

const LOCATION_UPDATE_INTERVAL = 10000; // ms
const LOCATION_CHANGE_THRESHOLD = 50; // metros
```

3. **Caché de Rutas**:
```javascript
const rutasCache = new Map();

const obtenerRuta = (origen, destino) => {
  const key = `${origen.lat},${origen.lng}-${destino.lat},${destino.lng}`;
  if (rutasCache.has(key)) {
    return rutasCache.get(key);
  }
  // Calcular ruta...
};
```

## 🚀 Despliegue

### Entorno de Desarrollo

```
┌──────────────────────────────┐
│     máquina del dev          │
├──────────────────────────────┤
│ - Expo CLI                   │
│ - Broker MQTT (Mosquitto)    │
│ - Emulador Android/iOS       │
│ - Node.js 18+                │
│ - npm/yarn                   │
└──────────────────────────────┘
```

### Entorno de Producción Recomendado

```
┌─────────────────────────────────────────┐
│         Producción                      │
├─────────────────────────────────────────┤
│                                         │
│  ┌──────────────────────┐              │
│  │  App Albergue (APK)  │              │
│  │  Google Play Store   │              │
│  └──────────────────────┘              │
│                                         │
│  ┌──────────────────────┐              │
│  │  App Repartidor      │              │
│  │  Google Play Store   │              │
│  └──────────────────────┘              │
│           ↓ (WebSocket)                │
│  ┌──────────────────────┐              │
│  │  Broker MQTT         │              │
│  │  (VPS/Cloud)         │              │
│  │  - SSL/TLS           │              │
│  │  - Autenticación     │              │
│  │  - Backups           │              │
│  │  - Monitoreo         │              │
│  └──────────────────────┘              │
│                                         │
│  ┌──────────────────────┐              │
│  │  Base de Datos       │              │
│  │  (Histórico)         │              │
│  └──────────────────────┘              │
│                                         │
└─────────────────────────────────────────┘
```

### Build para Producción (Android)

```bash
# 1. Crear build release
eas build --platform android --release

# 2. Generar APK/AAB firmado
./gradlew assembleRelease

# 3. Firmar APK
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 \
  -keystore my-release-key.keystore \
  app-release-unsigned.apk alias_name
```

---

**Última actualización**: Noviembre 2025
**Versión**: 1.0.0
