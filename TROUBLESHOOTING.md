# 🆘 Resolución de Problemas - Proyecto Albergue

Guía de troubleshooting para problemas comunes durante desarrollo y ejecución.

## 📋 Tabla de Contenidos

1. [Problemas de Instalación](#problemas-de-instalación)
2. [Problemas de Configuración](#problemas-de-configuración)
3. [Problemas de MQTT](#problemas-de-mqtt)
4. [Problemas de Ubicación](#problemas-de-ubicación)
5. [Problemas de Mapas](#problemas-de-mapas)
6. [Problemas de Rendimiento](#problemas-de-rendimiento)
7. [Problemas de Build/Deploy](#problemas-de-builddeploy)
8. [Debugging Avanzado](#debugging-avanzado)

## 📥 Problemas de Instalación

### npm install falla

**Error típico**:
```
npm ERR! code E404
npm ERR! 404 Not Found - GET https://registry.npmjs.org/paho-mqtt
```

**Soluciones**:
1. Verificar conexión a internet
2. Limpiar caché npm:
```bash
npm cache clean --force
npm cache verify
```

3. Cambiar registry si npm.js está caído:
```bash
npm config set registry https://registry.npmmirror.com
npm install
npm config set registry https://registry.npmjs.org/
```

### node_modules corrupto

**Síntomas**: Errores al importar módulos

**Solución**:
```bash
# En la carpeta del app
rm -r node_modules package-lock.json
npm install
```

### Versiones incompatibles

**Error**:
```
peer dep missing: react-native@0.81.5, installed: 0.80.5
```

**Solución**:
```bash
npm install react-native@0.81.5
npm audit fix
```

## ⚙️ Problemas de Configuración

### IP del Broker Incorrecta

**Síntoma**: "Cannot connect to broker"

**Verificar IP**:
```bash
# Windows
ipconfig | findstr "IPv4"

# macOS/Linux
ifconfig | grep "inet "
```

**Actualizar en App.js**:
```javascript
const BROKER_HOST = '192.168.1.137';  // Tu IP correcta
```

### Puerto Incorrecto

**Verificar disponibilidad**:
```bash
# Windows (PowerShell)
netstat -ano | findstr :9001

# macOS/Linux
lsof -i :9001
```

Si el puerto está en uso:
- Cambiar puerto en mosquitto.conf
- O cambiar puerto en App.js (debe coincidir)

### app.json mal configurado

**Revisar que contenga**:
```json
{
  "expo": {
    "name": "app-albergue",
    "slug": "app-albergue",
    "version": "1.0.0",
    "platforms": ["android", "ios", "web"],
    "android": {
      "package": "com.anonymous.appalbergue"
    }
  }
}
```

## 📡 Problemas de MQTT

### Broker no conecta

**Checklist**:
1. ¿Mosquitto está corriendo?
```bash
ps aux | grep mosquitto
sudo systemctl status mosquitto
docker ps | grep mosquitto
```

2. ¿Configuración correcta?
```bash
# Ver config
cat /etc/mosquitto/mosquitto.conf
```

3. ¿Puerto correcto?
```bash
# Puerto debe estar abierto
netstat -tuln | grep 9001
```

4. ¿Firewall bloqueando?
```bash
# Windows - Permitir puerto en Firewall
# macOS - Configuración → Seguridad
# Linux - sudo ufw allow 9001
```

### Timeout al conectar

**Síntoma**: Demora larga, luego error

**Solución**:
```javascript
client.connect({
  timeout: 10,  // Reducir timeout
  keepaliveInterval: 20
});
```

### Mensajes no se envían

**Verificar que MQTT esté conectado**:
```javascript
if (client && client.isConnected()) {
  // Enviar
} else {
  console.error('MQTT no conectado');
}
```

**Revisar consola para errores**:
```bash
# Ver logs de mosquitto
tail -f /var/log/mosquitto/mosquitto.log
```

### Desconexiones frecuentes

**Causa**: keepaliveInterval muy bajo

**Solución**:
```javascript
client.connect({
  keepaliveInterval: 60,  // Segundos
  timeout: 30
});
```

### QoS issues

**Problema**: Mensajes no llegan

**Verificar**:
```javascript
// En publicador
message.qos = 1;

// En suscriptor
client.subscribe(topic, { qos: 1 });
```

## 📍 Problemas de Ubicación

### Permiso Denegado

**Síntoma**: "PermissionError"

**Solución Android**:
1. Configuración → Aplicaciones → Permisos
2. Ubicación → Permitir durante uso de la app
3. Opcionalmente cambiar a "Siempre permitir"

**Solución iOS**:
1. Configuración → Privacidad → Localización
2. Buscar app → Seleccionar "Siempre" o "Al usar"

**En código - Solicitar nuevamente**:
```javascript
const { status } = await Location.requestForegroundPermissionsAsync();
if (status !== 'granted') {
  Alert.alert('Se requiere permiso de ubicación');
  return;
}
```

### Ubicación no se actualiza

**Causa 1**: GPS deshabilitado

**Solución**: Habilitar GPS en dispositivo

**Causa 2**: Precisión insuficiente

**Solución**:
```javascript
const location = await Location.getCurrentPositionAsync({
  accuracy: Location.Accuracy.Highest,
  timeout: 30000
});
```

**Causa 3**: WatchPosition no actualiza

```javascript
// Cambiar intervalo
const subscription = await Location.watchPositionAsync(
  {
    timeInterval: 3000,      // Cada 3 segundos
    distanceInterval: 5      // O cada 5 metros
  },
  handleLocationUpdate
);
```

### Ubicación no es precisa

**Síntomas**: Saltos grandes entre puntos

**Soluciones**:
1. Usar GPS en exterior (mejor señal)
2. Validar antes de usar:
```javascript
const { accuracy } = location.coords;
if (accuracy > 20) {  // Rechazar si > 20m
  console.warn('Precisión baja:', accuracy);
  return;
}
```

3. Suavizar datos:
```javascript
// Implementar filtro de Kalman o media móvil
const coordsFiltered = suavizarCoordenadas(coords);
```

## 🗺️ Problemas de Mapas

### WebView blanco

**Causa 1**: JavaScript deshabilitado

**Solución**:
```javascript
<WebView
  javaScriptEnabled={true}
  domStorageEnabled={true}
/>
```

**Causa 2**: OpenStreetMap no carga

**Solución**: Verificar internet, cambiar tile provider:
```javascript
// Alternativa a OpenStreetMap
L.tileLayer('https://maps.googleapis.com/maps/api/tile/{x}/{y}/{z}...', {
  attribution: 'Google Maps'
}).addTo(map);
```

**Causa 3**: Leaflet no carga

```javascript
// Verificar en HTML
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

### Markers no se muestran

**Solución 1**: Esperar a que WebView cargue
```javascript
<WebView onLoad={() => {
  refWebView.current?.injectJavaScript(script);
}}/>
```

**Solución 2**: Coordenadas inválidas
```javascript
// Validar antes
if (Math.abs(lat) <= 90 && Math.abs(lng) <= 180) {
  // Coordinates válidas
}
```

### Polyline no se muestra

**Causa**: Formato incorrecto de coordenadas

**Correcto**:
```javascript
const ruta = [
  [20.375, -99.982],  // [lat, lng]
  [20.376, -99.983]
];

L.polyline(ruta).addTo(map);
```

**Incorrecto**:
```javascript
// Esto NO funciona
const ruta = [
  {lat: 20.375, lng: -99.982}  // ❌
];
```

### Mapa muy zoom out

**Solución**: Usar fitBounds
```javascript
const bounds = rutaPolyline.getBounds();
map.fitBounds(bounds, { 
  padding: [50, 50],
  maxZoom: 15 
});
```

## 🚀 Problemas de Rendimiento

### App lenta al actualizar ubicación

**Causa**: Demasiadas actualizaciones

**Solución**: Throttling
```javascript
let lastUpdate = Date.now();
const MIN_INTERVAL = 5000; // 5 segundos

const actualizarUbicacion = (coords) => {
  const ahora = Date.now();
  if (ahora - lastUpdate < MIN_INTERVAL) return;
  
  lastUpdate = ahora;
  setRepartidorCoords(coords);
};
```

### WebView lento

**Causa**: Muchos puntos en polyline

**Solución**: Simplificar ruta
```javascript
const simplificarRuta = (ruta, tolerancia = 0.001) => {
  // Implementar Ramer-Douglas-Peucker algorithm
  return rutaSimplificada;
};
```

### Memory leak en MQTT

**Causa**: No limpiar suscripciones

**Solución**:
```javascript
useEffect(() => {
  const client = new Client(...);
  
  return () => {
    if (client.isConnected()) {
      client.unsubscribe(TOPIC_POSICION);
      client.disconnect();
    }
  };
}, []);
```

## 🏗️ Problemas de Build/Deploy

### Error al buildear para Android

**Causa**: Gradle incompatible

**Solución**:
```bash
cd app-albergue/android
./gradlew clean
./gradlew assembleDebug
```

### APK muy grande

**Causa**: Dependencias no optimizadas

**Solución**:
```gradle
android {
  packagingOptions {
    exclude 'lib/arm64-v8a/libc++_shared.so'
  }
}
```

### Instalación falla en dispositivo

**Causa 1**: Versión Android incompatible

Ver `build.gradle`:
```gradle
minSdkVersion 21
targetSdkVersion 34
```

**Causa 2**: Permisos no declarados

En `AndroidManifest.xml`:
```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.INTERNET" />
```

## 🔍 Debugging Avanzado

### Habilitar Logs Detallados

```javascript
// En App.js
const DEBUG = true;

const log = (tag, message) => {
  if (DEBUG) {
    console.log(`[${tag}] ${message}`);
  }
};

// Uso
log('MQTT', 'Intentando conectar...');
log('LOCATION', `Ubicación: ${lat}, ${lng}`);
```

### Usar Chrome DevTools

```bash
# Para WebView
adb shell 'echo > /proc/sys/net/ipv4/ip_forward'
adb reverse tcp:8081 tcp:8081
```

Luego abrir Chrome: `chrome://inspect`

### Debugging Expo

```bash
# Logs detallados
expo start --verbose

# Modo tunnel (evita firewall)
expo start --tunnel

# Seleccionar Android en terminal
# Presionar 'a' para abrir en emulador
```

### Debugging MQTT con mosquitto_sub

Terminal 1:
```bash
mosquitto_sub -h 192.168.1.137 -t "paquetes/#" -v
```

Terminal 2:
```bash
# Desde app repartidor
# Verá en Terminal 1:
# paquetes/posicion {"latitude": 20.375, ...}
```

### Inspeccionar Estado de App

```javascript
// En console
console.log('Estado actual:', {
  coords,
  albergueSeleccionado,
  repartidorCoords,
  eta,
  mqttConnected,
  viajeActivo
});
```

### Validar Mensajes MQTT

```javascript
const validarMensaje = (topic, data) => {
  console.log(`Mensaje de ${topic}:`, data);
  
  if (topic === 'paquetes/posicion') {
    console.assert(data.latitude !== undefined, 'Sin latitude');
    console.assert(data.longitude !== undefined, 'Sin longitude');
  }
  
  return true;
};
```

## 📞 Contacto para Problemas Adicionales

Si el problema persiste:

1. Verificar **README.md** y **TECHNICAL_DOCUMENTATION.md**
2. Revisar **logs completos** de Expo y Mosquitto
3. Abrir un issue en repositorio con:
   - Descripción del problema
   - Logs de error
   - SO y versiones
   - Pasos para reproducir

---

**Última actualización**: Noviembre 2025
**Versión**: 1.0.0
