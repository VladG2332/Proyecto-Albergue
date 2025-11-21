# 📚 Referencia de APIs - Proyecto Albergue

Documentación completa de componentes, funciones y APIs utilizadas.

## 📋 Tabla de Contenidos

1. [React Native API](#react-native-api)
2. [Expo Location API](#expo-location-api)
3. [Paho MQTT API](#paho-mqtt-api)
4. [WebView API](#webview-api)
5. [Leaflet API](#leaflet-api)
6. [React Hooks](#react-hooks)
7. [Estado Global y Contexto](#estado-global-y-contexto)

## 📱 React Native API

### Componentes Utilizados

#### View
```javascript
<View style={styles.container}>
  {/* Contenedor principal */}
</View>
```

**Props**:
- `style`: Estilos StyleSheet
- `onPress`: Manejador de click (en TouchableOpacity)

**Ejemplo**:
```javascript
<View style={{flex: 1, padding: 10}}>
  <Text>Contenido</Text>
</View>
```

#### Text
```javascript
<Text style={styles.titulo}>Título</Text>
```

**Props**:
- `style`: Estilos
- `numberOfLines`: Máximo de líneas
- `ellipsizeMode`: Cómo truncar ("tail", "head", "middle")

**Ejemplo**:
```javascript
<Text numberOfLines={1} ellipsizeMode="tail">
  Texto largo que se trunca...
</Text>
```

#### ScrollView
```javascript
<ScrollView style={styles.scroll}>
  {/* Contenido desplazable */}
</ScrollView>
```

**Props**:
- `style`: Estilos
- `horizontal`: Desplazamiento horizontal
- `scrollEventThrottle`: Frecuencia de eventos

**Ejemplo**:
```javascript
<ScrollView scrollEventThrottle={16}>
  {items.map(item => <ItemView key={item.id} />)}
</ScrollView>
```

#### TouchableOpacity
```javascript
<TouchableOpacity onPress={handlePress}>
  <Text>Presionar</Text>
</TouchableOpacity>
```

**Props**:
- `onPress`: Manejador de click
- `activeOpacity`: Opacidad al presionar (0-1)
- `disabled`: Deshabilitar botón

**Ejemplo**:
```javascript
<TouchableOpacity 
  onPress={() => alert('Presionado')}
  activeOpacity={0.7}
  disabled={isLoading}
>
  <Text>Click aquí</Text>
</TouchableOpacity>
```

#### TextInput
```javascript
<TextInput
  placeholder="Ingresa tu nombre"
  value={nombre}
  onChangeText={setNombre}
/>
```

**Props**:
- `placeholder`: Texto sugerencia
- `value`: Valor actual
- `onChangeText`: Callback cuando cambia
- `secureTextEntry`: Ocultar texto (contraseña)
- `editable`: Habilitado/Deshabilitado
- `maxLength`: Máximo de caracteres

**Ejemplo**:
```javascript
<TextInput
  placeholder="Nombre"
  value={nombre}
  onChangeText={setNombre}
  maxLength={50}
  style={{borderWidth: 1, padding: 10}}
/>
```

#### Alert
```javascript
Alert.alert('Título', 'Mensaje', [
  { text: 'Cancelar', onPress: () => {} },
  { text: 'Aceptar', onPress: () => {} }
]);
```

**Métodos**:
- `alert(title, message, buttons)`: Mostrar alerta
- `prompt(title, message, callback)`: Solicitar entrada

**Ejemplo**:
```javascript
Alert.alert(
  'Error',
  'No se pudo conectar al broker MQTT',
  [{ text: 'OK' }]
);
```

### StyleSheet

```javascript
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    paddingHorizontal: 15
  },
  button: {
    paddingVertical: 12,
    paddingHorizontal: 20,
    backgroundColor: '#3498db',
    borderRadius: 8
  }
});
```

**Propiedades Comunes**:
- `flex`: Crecimiento flexible
- `padding`: Espaciado interno
- `margin`: Espaciado externo
- `backgroundColor`: Color fondo
- `borderRadius`: Esquinas redondeadas
- `fontSize`: Tamaño fuente
- `color`: Color texto

## 📍 Expo Location API

### Tipos

```javascript
// Location.PermissionStatus
'undetermined' | 'granted' | 'denied'

// Location.LocationObject
{
  latitude: number,
  longitude: number,
  altitude: number,
  accuracy: number,
  altitudeAccuracy: number,
  heading: number,
  speed: number,
  timestamp: number
}
```

### Funciones Principales

#### requestForegroundPermissionsAsync()

```javascript
const { status } = await Location.requestForegroundPermissionsAsync();

if (status === 'granted') {
  // Permiso otorgado
}
```

**Retorna**: `{status: PermissionStatus, granted: boolean}`

**Errores**:
- `PermissionError`: Permiso rechazado
- `NotAvailableError`: Device no soporta localización

#### getCurrentPositionAsync(options)

```javascript
const location = await Location.getCurrentPositionAsync({
  accuracy: Location.Accuracy.Balanced,
  timeout: 30000
});

console.log(location.coords.latitude);
console.log(location.coords.longitude);
```

**Options**:
- `accuracy`: `Highest`, `High`, `Balanced`, `Low`
- `timeout`: Milisegundos
- `maximumAge`: Cache máximo

**Retorna**: `{coords: LocationObject, timestamp: number}`

#### watchPositionAsync(options, callback)

```javascript
const subscription = await Location.watchPositionAsync(
  {
    accuracy: Location.Accuracy.Balanced,
    timeInterval: 5000,
    distanceInterval: 10
  },
  (location) => {
    console.log(location.coords);
  }
);

// Detener
subscription.remove();
```

**Options**:
- `accuracy`: Precisión
- `timeInterval`: Milisegundos entre actualizaciones
- `distanceInterval`: Metros mínimos para actualizar

## 📡 Paho MQTT API

### Creación del Cliente

```javascript
const client = new Client(
  hostname,    // String: "192.168.1.137"
  port,        // Number: 9001
  path,        // String: "/mqtt"
  clientId     // String: "albergue-123"
);
```

### Propiedades

```javascript
client.isConnected()        // Boolean: Estado de conexión
client.clientId             // String: ID del cliente
client.onMessageArrived     // Function: Callback de mensajes
client.onConnectionLost     // Function: Callback desconexión
```

### Métodos de Conexión

#### connect(options)

```javascript
client.connect({
  useSSL: false,
  userName: "user",           // Opcional
  password: "pass",           // Opcional
  timeout: 30,                // Segundos
  keepaliveInterval: 60,      // Segundos
  cleanSession: true,         // Boolean
  onSuccess: () => {
    console.log('Conectado');
  },
  onFailure: (error) => {
    console.error('Error:', error);
  }
});
```

#### disconnect()

```javascript
if (client.isConnected()) {
  client.disconnect();
}
```

### Métodos de Publicación

#### subscribe(topic, options)

```javascript
client.subscribe('paquetes/posicion', {
  onSuccess: () => {
    console.log('Suscrito');
  },
  onFailure: (error) => {
    console.error('Error suscripción:', error);
  },
  qos: 1
});
```

#### send(message) / publish(message)

```javascript
const message = new Message('{"lat": 20.375}');
message.destinationName = 'paquetes/posicion';
message.qos = 0;

client.send(message);
```

### Message Class

```javascript
const message = new Message(payload);
message.destinationName = topic;
message.qos = 0;                    // 0, 1, 2
message.retained = false;
message.duplicate = false;

client.send(message);
```

### Callbacks

#### onMessageArrived

```javascript
client.onMessageArrived = (message) => {
  console.log(message.destinationName);  // Tema
  console.log(message.payloadString);    // Contenido
  console.log(message.qos);              // Calidad de servicio
  
  const data = JSON.parse(message.payloadString);
};
```

#### onConnectionLost

```javascript
client.onConnectionLost = (responseObject) => {
  console.log('Conexión perdida');
  console.log(responseObject.errorCode);
  console.log(responseObject.errorMessage);
};
```

## 🌐 WebView API

### Importación

```javascript
import { WebView } from 'react-native-webview';
```

### Componente WebView

```javascript
<WebView
  ref={refWebView}
  source={{ html: htmlContent }}
  originWhitelist={['*']}
  javaScriptEnabled={true}
  onLoad={() => console.log('Cargado')}
  onError={(syntheticEvent) => {
    const { nativeEvent } = syntheticEvent;
    console.error('WebView error:', nativeEvent);
  }}
/>
```

### Props

| Prop | Tipo | Descripción |
|------|------|------------|
| `source` | object | `{html: string}` o `{uri: string}` |
| `originWhitelist` | array | Orígenes permitidos |
| `javaScriptEnabled` | boolean | Habilitar JavaScript |
| `onLoad` | function | Callback cuando carga |
| `onError` | function | Callback de error |
| `style` | object | Estilos |
| `ref` | ref | Referencia al componente |

### Métodos

#### injectJavaScript(code)

```javascript
const refWebView = React.useRef();

refWebView.current.injectJavaScript(`
  window.updatePosicion({
    latitude: 20.375,
    longitude: -99.982
  });
`);
```

**Sintaxis**:
```javascript
refWebView.current.injectJavaScript(`
  // Código JavaScript a ejecutar
  console.log('Ejecutado en WebView');
  window.tuFuncion(parametros);
`);
```

## 🗺️ Leaflet API

### Inicialización

```javascript
const map = L.map('map').setView([20.38, -99.97], 14);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
  maxZoom: 19,
  attribution: '© OpenStreetMap contributors'
}).addTo(map);
```

### Markers

#### Crear Marker

```javascript
const marker = L.marker([20.375, -99.982], {
  title: 'Mi ubicación',
  icon: L.icon({
    iconUrl: 'url-a-imagen.png',
    iconSize: [32, 32],
    popupAnchor: [0, -16]
  })
}).addTo(map);

marker.bindPopup('📍 Ubicación');
marker.openPopup();
```

#### Actualizar Marker

```javascript
marker.setLatLng([20.376, -99.983]);
map.panTo([20.376, -99.983]);
```

#### Eliminar Marker

```javascript
map.removeLayer(marker);
```

### Polylines (Rutas)

#### Crear Polyline

```javascript
const routeCoords = [
  [20.375, -99.982],
  [20.376, -99.983],
  [20.377, -99.984]
];

const polyline = L.polyline(routeCoords, {
  color: '#2196F3',
  weight: 4,
  opacity: 0.8,
  dashArray: '5, 5'
}).addTo(map);
```

#### Ajustar Mapa a Ruta

```javascript
const bounds = polyline.getBounds();
map.fitBounds(bounds, { padding: [50, 50] });
```

#### Eliminar Polyline

```javascript
map.removeLayer(polyline);
```

### Control de Mapa

```javascript
// Centro y zoom
map.setView([20.375, -99.982], 15);

// Pan suave
map.panTo([20.376, -99.983]);

// Zoom
map.setZoom(14);
map.zoomIn();
map.zoomOut();
```

### Eventos

```javascript
map.on('click', (e) => {
  console.log('Coordenadas:', e.latlng.lat, e.latlng.lng);
});

marker.on('click', () => {
  console.log('Marker presionado');
});
```

## ⚛️ React Hooks

### useState

```javascript
const [state, setState] = useState(initialValue);

// Ejemplos
const [coords, setCoords] = useState(null);
const [count, setCount] = useState(0);
const [isVisible, setIsVisible] = useState(false);
```

**Patrón de actualización**:
```javascript
setState(newValue);           // Reemplazar
setState(prev => prev + 1);   // Basado en anterior
```

### useEffect

```javascript
useEffect(() => {
  // Código a ejecutar
  
  return () => {
    // Cleanup (opcional)
  };
}, [dependencies]);
```

**Ejemplos**:

```javascript
// Se ejecuta una sola vez (al montar)
useEffect(() => {
  initializeApp();
}, []);

// Se ejecuta cuando cambia 'viajeActivo'
useEffect(() => {
  if (viajeActivo) {
    connectToMQTT();
  }
}, [viajeActivo]);

// Se ejecuta después de cada render
useEffect(() => {
  updateUI();
});

// Cleanup
useEffect(() => {
  const subscription = addEventListener();
  
  return () => {
    removeEventListener(subscription);
  };
}, []);
```

### useRef

```javascript
const refWebView = React.useRef();

// Usar
refWebView.current.injectJavaScript(code);
```

## 🗄️ Estado Global y Contexto

### Actualmente en Proyecto

El proyecto **NO utiliza Context API ni Redux**. Todo el estado está en el componente principal `App.js`.

**Estados App-Albergue**:
```javascript
const [coords, setCoords] = useState(null);
const [albergueSeleccionado, setAlbergueSeleccionado] = useState(null);
const [repartidorCoords, setRepartidorCoords] = useState(null);
const [repartidorNombre, setRepartidorNombre] = useState('Repartidor');
const [eta, setEta] = useState(null);
const [ruta, setRuta] = useState(null);
const [lastUpdate, setLastUpdate] = useState(null);
const [mqttClient, setMqttClient] = useState(null);
const [mqttConnected, setMqttConnected] = useState(false);
const [viajeActivo, setViajeActivo] = useState(false);
```

### Mejora Futura: Context API

Para refactorizar estado a nivel global:

```javascript
// AlbergueContext.js
export const AlbergueContext = React.createContext();

export function AlbergueProvider({ children }) {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <AlbergueContext.Provider value={{state, dispatch}}>
      {children}
    </AlbergueContext.Provider>
  );
}

// En App.js
import { useContext } from 'react';
const { state, dispatch } = useContext(AlbergueContext);
```

---

**Última actualización**: Noviembre 2025
**Versión**: 1.0.0
