# 🔧 Documentación Técnica - Proyecto Albergue

Referencia técnica detallada para desarrolladores. Incluye arquitectura, componentes, estados y funciones.

## 📋 Tabla de Contenidos

1. [Stack Tecnológico](#stack-tecnológico)
2. [Estructura de Directorios](#estructura-de-directorios)
3. [Dependencias Explicadas](#dependencias-explicadas)
4. [Componentes Principales](#componentes-principales)
5. [States (Estados) y Hooks](#states-estados-y-hooks)
6. [Effects (Efectos)](#effects-efectos)
7. [Funciones Clave](#funciones-clave)
8. [Configuración Expo](#configuración-expo)
9. [Temas MQTT](#temas-mqtt)
10. [Manejo de Errores](#manejo-de-errores)

## 🏗️ Stack Tecnológico

### Versiones Principales

| Tecnología | Versión | Uso |
|-----------|---------|-----|
| React | 19.1.0 | Framework UI |
| React Native | 0.81.5 | Framework multiplataforma |
| Expo | ~54.0.23 | Plataforma de desarrollo |
| Paho MQTT | 1.1.0 | Cliente MQTT |
| Leaflet | 1.9.4 | Mapas interactivos |
| Axios | 1.13.2 | Cliente HTTP |

### Arquitectura General

```
┌────────────────────────────────────────────────────────┐
│                    CAPA PRESENTACIÓN                   │
│  (React Components - App.js)                           │
├────────────────────────────────────────────────────────┤
│  - Views (ScrollView, Modal, WebView)                  │
│  - Touchable Components                                │
│  - TextInput                                           │
└────────────────────────────────────────────────────────┘
                          ↓
┌────────────────────────────────────────────────────────┐
│                  CAPA DE LÓGICA                        │
│  (React Hooks - useState, useEffect)                   │
├────────────────────────────────────────────────────────┤
│  - Estados (coords, ruta, eta, etc)                    │
│  - Efectos (MQTT, WebView, Location)                   │
│  - Funciones de negocio                                │
└────────────────────────────────────────────────────────┘
                          ↓
┌────────────────────────────────────────────────────────┐
│                   CAPA DE DATOS                        │
│  (APIs Nativas y Externas)                             │
├────────────────────────────────────────────────────────┤
│  - expo-location (GPS)                                 │
│  - Paho MQTT (Comunicación)                            │
│  - WebView (Mapas HTML)                                │
└────────────────────────────────────────────────────────┘
                          ↓
┌────────────────────────────────────────────────────────┐
│              CAPA DE SERVICIOS EXTERNOS                │
├────────────────────────────────────────────────────────┤
│  - Broker MQTT (192.168.1.137:9001)                    │
│  - OpenStreetMap (Tiles de mapas)                      │
│  - Leaflet CDN (Librerías frontend)                    │
└────────────────────────────────────────────────────────┘
```

## 📁 Estructura de Directorios

### App-Albergue

```
app-albergue/
├── App.js                      # Componente principal (700+ líneas)
├── app.json                    # Configuración Expo
├── index.js                    # Punto de entrada
├── package.json                # Dependencias y scripts
├── package-lock.json           # Lock file
├── android/
│   ├── app/
│   │   ├── src/main/
│   │   │   ├── AndroidManifest.xml
│   │   │   ├── java/com/anonymous/appalbergue/
│   │   │   │   ├── MainActivity.kt
│   │   │   │   └── MainApplication.kt
│   │   │   └── res/              # Recursos (drawables, valores)
│   │   └── build.gradle
│   ├── build.gradle
│   ├── gradle.properties
│   ├── settings.gradle
│   └── gradlew
├── .expo/                      # Config de Expo
├── .gitignore
└── assets/                     # Imágenes, fuentes, etc

Tamaño aproximado: ~500KB (sin node_modules)
Dependencias: ~15 paquetes directos
```

### App-Repartidor

```
app-repartidor/
├── App.js                      # Componente principal (500+ líneas)
├── app.json                    # Configuración Expo
├── index.js                    # Punto de entrada
├── package.json                # Dependencias y scripts
├── package-lock.json           # Lock file
├── android/
│   ├── app/
│   │   ├── src/main/
│   │   │   ├── AndroidManifest.xml
│   │   │   ├── java/com/anonymous/apprepartidor/
│   │   │   │   ├── MainActivity.kt
│   │   │   │   └── MainApplication.kt
│   │   │   └── res/              # Recursos
│   │   └── build.gradle
│   ├── build.gradle
│   ├── gradle.properties
│   ├── settings.gradle
│   └── gradlew
├── .expo/
├── .gitignore
└── assets/                     # Imágenes, fuentes, etc

Tamaño aproximado: ~500KB (sin node_modules)
```

## 📦 Dependencias Explicadas

### Dependencias Principales

#### 1. **expo** (~54.0.23)
```json
"expo": "~54.0.23"
```
- Plataforma de desarrollo y tiempo de ejecución
- Maneja build para Android/iOS
- Proporciona APIs nativas (Location, Permissions)
- Sirve la aplicación en modo desarrollo

#### 2. **react** (19.1.0) & **react-native** (0.81.5)
```json
"react": "19.1.0",
"react-native": "0.81.5"
```
- Framework base para UI multiplataforma
- Componentes nativos (View, Text, ScrollView, etc)
- Sistema reactivo con hooks

#### 3. **expo-location** (^19.0.7)
```json
"expo-location": "^19.0.7"
```
- API para acceder a GPS del dispositivo
- Métodos: `requestForegroundPermissionsAsync()`, `getCurrentPositionAsync()`
- Proporciona precisión y altitud

#### 4. **paho-mqtt** (^1.1.0)
```json
"paho-mqtt": "^1.1.0"
```
- Cliente MQTT para JavaScript
- Soporta WebSocket
- Métodos: `connect()`, `subscribe()`, `send()`, `disconnect()`

#### 5. **react-native-webview** (13.15.0)
```json
"react-native-webview": "13.15.0"
```
- Componente para renderizar HTML/JavaScript
- Cargará el mapa Leaflet.js
- Inyección de JavaScript: `injectJavaScript()`

#### 6. **axios** (^1.13.2)
```json
"axios": "^1.13.2"
```
- Cliente HTTP (actualmente no utilizado, pero disponible para futuros servicios)

#### 7. **react-native-maps** (^1.26.18)
```json
"react-native-maps": "^1.26.18"
```
- Componente de mapas nativo (alternativa a WebView)
- Actualmente no se utiliza, pero está disponible

### Dependencias de Desarrollo

```json
"devDependencies": {
  "expo-module-scripts": "^5.0.7"
}
```
- Herramientas de compilación y scripting para módulos Expo

## 🧩 Componentes Principales

### Componente App (App-Albergue)

**Propósito**: Componente raíz que gestiona toda la lógica de seguimiento de albergue.

**Estructura Básica**:
```javascript
export default function App() {
  // Estados
  const [coords, setCoords] = useState(null);
  // ... más estados
  
  // HTML del mapa
  const mapaHTML = `...`;
  
  // Efectos
  useEffect(() => { ... }, [viajeActivo]);
  // ... más efectos
  
  // Funciones
  const obtenerUbicacionActual = async () => { ... };
  const seleccionarAlbergue = (albergue) => { ... };
  const calcularETAYRuta = async (posRepartidor, clientMqtt) => { ... };
  
  // Render condicional
  return (
    viajeActivo ? <MapView /> : <MenuView />
  );
}
```

**Flujo de Renderizado**:
```
App Component
├── if (!viajeActivo)
│   └── MenuView
│       ├── Text (Título)
│       ├── ScrollView
│       │   ├── AlbergueButton[]
│       │   └── UbicacionButton
│       └── TouchableOpacity (Acciones)
│
└── if (viajeActivo)
    └── MapView
        ├── WebView (Leaflet Map)
        ├── ModalOverlay
        │   ├── CloseButton
        │   ├── MqttStatus
        │   ├── AlbergueInfo
        │   ├── RepartidorInfo
        │   └── EtaDisplay
        └── StyleSheet (Estilos)
```

## 🔌 States (Estados) y Hooks

### Estados en App-Albergue

| Estado | Tipo | Inicial | Descripción |
|--------|------|---------|------------|
| `coords` | object | null | Coordenadas del albergue seleccionado `{latitude, longitude}` |
| `albergueSeleccionado` | object | null | Datos del albergue `{id, nombre, lat, lng}` |
| `repartidorCoords` | object | null | Ubicación actual del repartidor `{latitude, longitude}` |
| `repartidorNombre` | string | 'Repartidor' | Nombre del repartidor |
| `eta` | number | null | Minutos estimados de llegada |
| `ruta` | array | null | Array de coordenadas `[[lat,lng], ...]` |
| `lastUpdate` | string | null | Timestamp del última actualización |
| `mqttClient` | object | null | Instancia del cliente MQTT |
| `mqttConnected` | boolean | false | Estado de conexión MQTT |
| `viajeActivo` | boolean | false | Si hay sesión activa |

### Estados en App-Repartidor

| Estado | Tipo | Inicial | Descripción |
|--------|------|---------|------------|
| `coords` | object | null | Ubicación actual del repartidor |
| `ruta` | array | null | Array de puntos de ruta |
| `destino` | object | null | Ubicación del albergue destino |
| `eta` | number | null | Minutos estimados |
| `nombre` | string | '' | Nombre del repartidor |
| `mqttClient` | object | null | Cliente MQTT |
| `mqttConnected` | boolean | false | Estado MQTT |
| `webViewReady` | boolean | false | WebView cargado |
| `viajeActivo` | boolean | false | Sesión activa |

## ⚡ Effects (Efectos)

### Efecto Principal MQTT (App-Albergue)

```javascript
useEffect(() => {
  if (!viajeActivo) return;
  
  // 1. Crear cliente MQTT
  const client = new Client(BROKER_HOST, BROKER_PORT, BROKER_PATH, 
                           'albergue-' + Math.random());
  
  // 2. Configurar handlers
  client.onMessageArrived = (message) => {
    // Procesar mensajes
  };
  
  client.onConnectionLost = (responseObject) => {
    // Manejar desconexión
  };
  
  // 3. Conectar
  client.connect({
    useSSL: false,
    onSuccess: () => {
      client.subscribe(TOPIC_POSICION);
      setMqttConnected(true);
    },
    onFailure: (e) => {
      Alert.alert('Error MQTT', 'No se pudo conectar');
    }
  });
  
  // 4. Cleanup
  return () => {
    if (client.isConnected()) {
      client.disconnect();
    }
  };
}, [viajeActivo]);
```

**Dependencias**: `[viajeActivo]`
**Se ejecuta**: Cuando cambia `viajeActivo` de false a true

### Efecto de Actualización de Marcador

```javascript
useEffect(() => {
  if (refWebView.current && albergueSeleccionado) {
    const script = `window.setAlbergue(${albergueSeleccionado.lat}, 
                                       ${albergueSeleccionado.lng});`;
    refWebView.current.injectJavaScript(script);
  }
}, [albergueSeleccionado]);
```

**Propósito**: Inyectar JavaScript en WebView cuando cambia el albergue

### Efecto de Actualización de Posición

```javascript
useEffect(() => {
  if (refWebView.current && repartidorCoords) {
    const script = `window.updatePosicion(${JSON.stringify(repartidorCoords)});`;
    refWebView.current.injectJavaScript(script);
  }
}, [repartidorCoords]);
```

**Propósito**: Actualizar marcador del repartidor en el mapa

## 🎯 Funciones Clave

### obtenerUbicacionActual() - App-Albergue

```javascript
const obtenerUbicacionActual = async () => {
  // 1. Solicitar permiso
  const { status } = await Location.requestForegroundPermissionsAsync();
  if (status !== 'granted') {
    Alert.alert('Permiso denegado');
    return;
  }
  
  // 2. Obtener ubicación
  try {
    const location = await Location.getCurrentPositionAsync({});
    setCoords(location.coords);
    setAlbergueSeleccionado({
      nombre: 'Mi Ubicación',
      lat: location.coords.latitude,
      lng: location.coords.longitude
    });
    setViajeActivo(true);
  } catch (err) {
    Alert.alert('Error', 'No se pudo obtener ubicación');
  }
};
```

**Parámetros**: Ninguno
**Retorna**: void (actualiza estados)
**Errores**: Permisos denegados, fallo de GPS

### calcularETAYRuta() - App-Albergue

```javascript
const calcularETAYRuta = async (posRepartidor, clientMqtt) => {
  // 1. Validar datos
  const destino = { lat: coords.latitude, lng: coords.longitude };
  const origen = { lat: posRepartidor.latitude, lng: posRepartidor.longitude };
  
  // 2. Calcular distancia (km)
  const latDiff = destino.lat - origen.lat;
  const lngDiff = destino.lng - origen.lng;
  const distancia = Math.sqrt(latDiff * latDiff + lngDiff * lngDiff) * 111;
  
  // 3. Calcular ETA (minutos a 50 km/h)
  const minutos = Math.ceil(distancia / 0.8);
  setEta(minutos);
  
  // 4. Interpolar puntos de ruta
  const rutaCoords = [];
  const numPuntos = Math.min(50, Math.max(10, Math.floor(distancia / 0.5)));
  
  for (let i = 0; i <= numPuntos; i++) {
    const t = i / numPuntos;
    const lat = origen.lat + (destino.lat - origen.lat) * t;
    const lng = origen.lng + (destino.lng - origen.lng) * t;
    rutaCoords.push([lat, lng]);
  }
  setRuta(rutaCoords);
  
  // 5. Enviar ruta a repartidor (MQTT)
  if (clientMqtt && clientMqtt.isConnected()) {
    const mensaje = {
      ruta: rutaCoords,
      destino: { lat: destino.lat, lng: destino.lng },
      eta: minutos
    };
    const message = new Message(JSON.stringify(mensaje));
    message.destinationName = TOPIC_RUTA;
    clientMqtt.send(message);
  }
};
```

**Parámetros**:
- `posRepartidor`: Objeto con `latitude` y `longitude`
- `clientMqtt`: Instancia del cliente MQTT

**Retorna**: Promise (async)

**Lógica**:
1. Calcula distancia euclidiana
2. Estima tiempo (distancia / 0.8 km/min ≈ 50 km/h)
3. Interpola 10-50 puntos entre origen y destino
4. Crea polyline suave
5. Envía ruta y destino al repartidor

### seleccionarAlbergue() - App-Albergue

```javascript
const seleccionarAlbergue = (albergue) => {
  setCoords({ latitude: albergue.lat, longitude: albergue.lng });
  setAlbergueSeleccionado(albergue);
  setViajeActivo(true);
};
```

**Parámetros**: 
- `albergue`: Objeto `{id, nombre, lat, lng}`

**Retorna**: void

## ⚙️ Configuración Expo

### app.json (App-Albergue)

```json
{
  "expo": {
    "name": "app-albergue",
    "slug": "app-albergue",
    "version": "1.0.0",
    "assetBundlePatterns": ["**/*"],
    "platforms": ["android", "ios", "web"],
    "android": {
      "package": "com.anonymous.appalbergue",
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      }
    },
    "plugins": [
      [
        "expo-location",
        {
          "locationAlwaysAndWhenInUsePermission": "Allow $(PRODUCT_NAME) to access your location."
        }
      ]
    ]
  }
}
```

**Configuraciones Clave**:
- `slug`: Identificador único
- `platforms`: Plataformas soportadas
- `android.package`: Nombre del paquete Java
- `plugins`: Expo plugins requeridos (location)

## 📡 Temas MQTT

### Estructura de Mensajes

#### Tema: `paquetes/posicion`

**Dirección**: Repartidor → Albergue

**Formato**:
```json
{
  "latitude": 20.375264824639682,
  "longitude": -99.9821426911384,
  "timestamp": 1700000000000,
  "precision": 5.0,
  "altitude": 2250.5
}
```

**Frecuencia**: Cada 5 segundos

**QoS**: 1 (At least once)

#### Tema: `paquetes/ruta`

**Dirección**: Albergue → Repartidor

**Formato**:
```json
{
  "ruta": [
    [20.375, -99.982],
    [20.376, -99.983],
    [20.377, -99.984]
  ],
  "destino": {
    "lat": 20.389,
    "lng": -99.997
  },
  "eta": 15,
  "timestamp": 1700000000000,
  "nombre_albergue": "Albergue Tecnologico"
}
```

**Frecuencia**: Cuando se calcula (cuando llega ubicación del repartidor)

**QoS**: 1 (At least once)

## ❌ Manejo de Errores

### Errores de Conexión MQTT

```javascript
client.connect({
  onSuccess: () => { /* ... */ },
  onFailure: (e) => {
    console.error('Error conectando:', e);
    console.error('Código:', e.errorCode);
    console.error('Mensaje:', e.errorMessage);
    
    Alert.alert('Error MQTT', 
      `Broker no disponible: ${BROKER}`);
  }
});
```

**Causas Comunes**:
- Broker no corriendo
- IP/Puerto incorrecto
- Firewall bloqueando puerto
- WebSocket no habilitado

### Errores de Localización

```javascript
try {
  const location = await Location.getCurrentPositionAsync({});
} catch (err) {
  // PERMISSION_DENIED
  // POSITION_UNAVAILABLE
  // TIMEOUT
  Alert.alert('Error', 'No se pudo obtener ubicación');
}
```

**Causas Comunes**:
- Permiso no otorgado
- GPS deshabilitado
- Interiores sin señal

### Errores de WebView

```javascript
<WebView
  onError={(syntheticEvent) => {
    const { nativeEvent } = syntheticEvent;
    console.warn('WebView error:', nativeEvent);
  }}
  onLoad={() => {
    // WebView cargó exitosamente
  }}
/>
```

## 🔍 Debugging

### Console Logs Implementados

```javascript
// MQTT
console.log('✅ Albergue conectado a MQTT');
console.error('❌ Error conectando a MQTT');

// Ubicación
console.log('🗺️ Calculando ruta simplificada');
console.log('Origen (Repartidor):', origen);

// Mapa
console.log('✅ WebView cargado');
console.log('Primeras 3 coordenadas:', rutaCoords.slice(0, 3));

// Envío de datos
console.log('✅ Ruta simplificada enviada al repartidor');
```

### Verificar Estado MQTT

En la app, el estado MQTT se muestra:
- Verde ✅ si `mqttConnected === true`
- Rojo ❌ si `mqttConnected === false`

### Verificar Ubicación

En Android:
1. Configuración → Aplicaciones → App Albergue → Permisos
2. Habilitar "Ubicación"

En iOS:
1. Configuración → Privacidad → Localización
2. Buscar app y seleccionar "Siempre" o "Al usar la app"

---

**Última actualización**: Noviembre 2025
**Versión**: 1.0.0
