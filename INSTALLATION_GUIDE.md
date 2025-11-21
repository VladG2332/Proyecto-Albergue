# 🚀 Guía de Instalación Detallada - Proyecto Albergue

Instrucciones paso a paso para instalar, configurar y ejecutar el proyecto.

## 📋 Tabla de Contenidos

1. [Requisitos Previos](#requisitos-previos)
2. [Instalación del Broker MQTT](#instalación-del-broker-mqtt)
3. [Instalación del Proyecto](#instalación-del-proyecto)
4. [Configuración](#configuración)
5. [Pruebas](#pruebas)
6. [Despliegue en Dispositivo](#despliegue-en-dispositivo)
7. [Troubleshooting](#troubleshooting)

## ✅ Requisitos Previos

### Software Requerido

#### Node.js y npm

**Verificar instalación**:
```bash
node --version    # Debe ser >= 18.0.0
npm --version     # Debe ser >= 9.0.0
```

**Descargar**: https://nodejs.org/

#### Git

**Verificar**:
```bash
git --version     # Debe mostrar versión
```

**Descargar**: https://git-scm.com/

#### Expo CLI

```bash
npm install -g expo-cli

# Verificar
expo --version    # Debe mostrar versión
```

#### Android Studio (Opcional, para emulador)

**Descargar**: https://developer.android.com/studio

**Configuración**:
1. Instalar Android Studio
2. Descargar SDK Android (nivel 31+)
3. Configurar emulador virtual

#### Editor de Código

**Recomendado**: Visual Studio Code
- Descargar: https://code.visualstudio.com/
- Extensiones útiles:
  - ES7+ React/Redux/React-Native snippets
  - Prettier
  - ESLint

### Hardware Requerido

- **Computadora**: Windows, macOS o Linux
- **Emulador/Dispositivo**: Android 8.0+ o iOS 12.0+
- **Red**: Acceso a red local con el broker MQTT

## 🏗️ Instalación del Broker MQTT

### Opción 1: Mosquitto en Windows (WSL)

#### Paso 1: Instalar WSL

```powershell
wsl --install
```

#### Paso 2: Instalar Mosquitto en WSL

```bash
sudo apt-get update
sudo apt-get install mosquitto mosquitto-clients
```

#### Paso 3: Configurar Mosquitto

Crear archivo de configuración:

```bash
sudo nano /etc/mosquitto/mosquitto.conf
```

Agregar/modificar:

```conf
port 1883
listener 9001
protocol websockets
allow_anonymous true
persistence true
persistence_location /var/lib/mosquitto/
```

#### Paso 4: Iniciar Mosquitto

```bash
sudo systemctl start mosquitto
sudo systemctl status mosquitto

# Para que inicie automáticamente
sudo systemctl enable mosquitto
```

### Opción 2: Docker

#### Instalar Docker

Descargar desde: https://www.docker.com/

#### Descargar Imagen Mosquitto

```bash
docker pull eclipse-mosquitto
```

#### Crear Contenedor

```bash
docker run -d \
  --name mosquitto \
  -p 1883:1883 \
  -p 9001:9001 \
  eclipse-mosquitto
```

#### Verificar Estado

```bash
docker ps                    # Ver contenedores activos
docker logs mosquitto        # Ver logs
docker stop mosquitto        # Detener
docker start mosquitto       # Iniciar
```

### Opción 3: Mosquitto en macOS

```bash
# Instalar con Homebrew
brew install mosquitto

# Iniciar servicio
brew services start mosquitto

# Verificar
brew services list
```

### Verificar Instalación MQTT

```bash
# Terminal 1: Suscribirse
mosquitto_sub -h localhost -t "test/topic"

# Terminal 2: Publicar
mosquitto_pub -h localhost -t "test/topic" -m "Hola"

# Debería ver "Hola" en Terminal 1
```

## 📦 Instalación del Proyecto

### Paso 1: Clonar Repositorio

```bash
git clone <url-del-repositorio>
cd proyecto-albergue
```

### Paso 2: Instalar App Albergue

```bash
cd app-albergue

# Instalar dependencias
npm install

# Verificar instalación
npm list

# Resultado esperado:
# ├── expo@54.0.23
# ├── react@19.1.0
# ├── react-native@0.81.5
# ├── paho-mqtt@1.1.0
# └── ...
```

### Paso 3: Instalar App Repartidor

```bash
cd ../app-repartidor

npm install

npm list
```

### Paso 4: Verificar Estructura

```
proyecto-albergue/
├── app-albergue/
│   ├── node_modules/        ✅ (creado por npm)
│   ├── package.json
│   └── App.js
├── app-repartidor/
│   ├── node_modules/        ✅ (creado por npm)
│   ├── package.json
│   └── App.js
└── README.md
```

## ⚙️ Configuración

### Configurar IP del Broker

#### Encontrar IP Local

**En Windows (PowerShell)**:
```powershell
ipconfig

# Buscar "IPv4 Address" en tu adaptador de red
# Ejemplo: 192.168.1.137
```

**En macOS/Linux**:
```bash
ifconfig

# Buscar "inet" en tu interfaz de red
# Ejemplo: 192.168.1.137
```

#### Actualizar App-Albergue

En `app-albergue/App.js`:

```javascript
// Línea ~10
const BROKER = 'ws://192.168.1.137:9001';
const BROKER_HOST = '192.168.1.137';
```

Reemplazar `192.168.1.137` con tu IP.

#### Actualizar App-Repartidor

En `app-repartidor/App.js`:

```javascript
// Línea ~10
const BROKER = 'ws://192.168.1.137:9001';
const BROKER_HOST = '192.168.1.137';
```

Misma IP que arriba.

### Verificar Conectividad

```bash
# Probar conexión al broker
ping 192.168.1.137

# Probar puerto WebSocket
nc -zv 192.168.1.137 9001
```

## 🧪 Pruebas

### Prueba 1: Verificar MQTT Conecta

```bash
# Terminal 1: Suscribir
mosquitto_sub -h 192.168.1.137 -t "paquetes/#"

# Terminal 2: Publicar
mosquitto_pub -h 192.168.1.137 -t "paquetes/posicion" \
  -m '{"latitude": 20.375, "longitude": -99.982}'

# Debería ver el mensaje en Terminal 1
```

### Prueba 2: Iniciar App en Desarrollo

#### App-Albergue

```bash
cd app-albergue
npm start

# Salida esperada:
# › Metro Bundler ready
# › Scan with the camera app to view the development server QR code
```

#### Escanear con Expo Go

1. Descargar app "Expo Go" en Play Store
2. Abrir aplicación
3. Escanear código QR con cámara
4. App debería cargar

#### Probar Funcionalidad

1. Presionar botón de albergue
2. Debería conectarse a MQTT
3. Esperaría ubicación del repartidor

### Prueba 3: Iniciar App Repartidor

```bash
cd ../app-repartidor
npm start
```

Escanear QR con Expo Go.

#### Probar Flujo Completo

1. **App Repartidor**:
   - Ingresar nombre
   - Presionar "Iniciar Viaje"
   - Otorgar permiso de ubicación

2. **App Albergue**:
   - Seleccionar albergue
   - Esperar ubicación del repartidor
   - Ver ubicación actualizada en mapa

## 📱 Despliegue en Dispositivo

### Opción 1: Emulador Android

#### Abrir Emulador

```bash
# Ver emuladores disponibles
emulator -list-avds

# Iniciar emulador
emulator -avd <nombre_emulador>
```

#### Instalar App

```bash
cd app-albergue
npm run android

# O directamente con Expo
expo run:android
```

### Opción 2: Dispositivo Físico Android

#### Preparar Dispositivo

1. Conectar dispositivo con cable USB
2. Activar "Depuración USB" en Configuración → Opciones de Desarrollador
3. Verificar conexión:

```bash
adb devices

# Debería mostrar:
# List of attached devices
# XXXXXX device
```

#### Instalar App

```bash
cd app-albergue

# Opción A: Expo
expo run:android

# Opción B: Gradle
./gradlew installDebug
```

### Opción 3: Build para Producción

#### Crear APK Firmado

```bash
cd app-albergue

# Generar keystore
keytool -genkey -v -keystore my-release-key.keystore \
  -alias my-key-alias \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000

# Build release
./gradlew assembleRelease

# Firmar APK
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 \
  -keystore my-release-key.keystore \
  app/build/outputs/apk/release/app-release-unsigned.apk \
  my-key-alias
```

#### Subir a Google Play Store

1. Crear cuenta Google Play Developer ($25 usd)
2. Crear proyecto nuevo
3. Subir APK firmado
4. Completar información y enviar a revisión

## 🔧 Troubleshooting

### Problema: "Cannot find module 'paho-mqtt'"

**Solución**:
```bash
cd app-albergue
npm install paho-mqtt --save
npm list paho-mqtt
```

### Problema: "MQTT Connection Refused"

**Causas posibles**:
1. Broker no está corriendo
2. IP del broker es incorrecta
3. Puerto 9001 está bloqueado

**Soluciones**:
```bash
# Verificar broker activo
ps aux | grep mosquitto

# Verificar puerto
netstat -an | grep 9001

# Reiniciar broker
sudo systemctl restart mosquitto
```

### Problema: "Permission Denied" Localización

**Solución**: En dispositivo:
1. Configuración → Aplicaciones → App Albergue
2. Permisos → Ubicación → Seleccionar "Permitir"

### Problema: WebView Mapas Blanco

**Solución**:
1. Verificar JavaScript habilitado en WebView
2. Verificar conexión a internet (para OpenStreetMap)
3. Revisar logs en `adb logcat`

### Problema: "npm ERR! Unexpected end of JSON input"

**Solución**:
```bash
npm cache clean --force
rm package-lock.json
npm install
```

### Problema: Emulador Android Lento

**Soluciones**:
```bash
# Asignar más memoria
emulator -avd <nombre> -memory 2048 -gpu on

# O usar aceleración KVM (Linux)
emulator -avd <nombre> -accel on
```

### Problema: Expo no conecta a red local

**Causa**: Firewall bloqueando puertos Expo

**Solución**:
```bash
# Usar tunnel en lugar de LAN
expo start --tunnel
```

## 📋 Checklist Final

Antes de considerar instalación completa:

- [ ] Node.js 18+ instalado
- [ ] npm 9+ instalado
- [ ] Git instalado
- [ ] Expo CLI instalado
- [ ] Broker MQTT corriendo
- [ ] IP del broker configurada en Apps
- [ ] npm install completado en ambas apps
- [ ] Expo Go descargado en dispositivo
- [ ] App Albergue carga sin errores
- [ ] App Repartidor carga sin errores
- [ ] MQTT conecta desde apps
- [ ] WebView (mapas) carga correctamente
- [ ] Localización solicitada y otorgada

---

**Última actualización**: Noviembre 2025
**Versión**: 1.0.0
