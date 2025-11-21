# 📦 Proyecto Albergue - Sistema de Tracking de Paquetes

Sistema de seguimiento en tiempo real para la distribución de paquetes entre repartidores y albergues, utilizando tecnología MQTT para comunicación bidireccional y React Native para el desarrollo multiplataforma.

## 📋 Tabla de Contenidos

- [Descripción General](#descripción-general)
- [Características](#características)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Requisitos Previos](#requisitos-previos)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Uso](#uso)
- [Tecnologías](#tecnologías)
- [Documentación](#documentación)
- [Contribución](#contribución)
- [Licencia](#licencia)

## 🎯 Descripción General

**Proyecto Albergue** es una aplicación de seguimiento logístico que permite:

- **Para Albergues**: Recibir ubicación en tiempo real de repartidores y calcular ETA automáticamente
- **Para Repartidores**: Recibir rutas optimizadas, destinos y tiempos estimados

El sistema utiliza MQTT como protocolo de comunicación para garantizar eficiencia, baja latencia y confiabilidad en la transmisión de datos.

### Casos de Uso Principales

1. **Gestor de Albergue**: 
   - Selecciona su albergue o proporciona ubicación actual
   - Visualiza ubicación del repartidor en tiempo real
   - Recibe estimación de tiempo de llegada (ETA)
   - Monitorea movimiento mediante mapa interactivo

2. **Repartidor**:
   - Se conecta y comparte su ubicación periódicamente
   - Recibe ruta optimizada al albergue destino
   - Visualiza destino en el mapa
   - Sigue guía visual para completar entrega

## ✨ Características

### 🗺️ Geolocalización y Mapas
- Maps interactivos con Leaflet.js
- Markers personalizados con iconos
- Visualización de rutas en tiempo real
- Cálculo automático de distancias
- Centrado automático en ubicación

### 🔄 Comunicación MQTT
- Protocolo pub/sub eficiente
- Baja latencia
- Reconexión automática
- Temas estructurados para datos

### 📱 Interfaz Responsiva
- Menú de selección de albergues
- Modal inferior con información
- Cierre de sesión fluido
- Diseño Android-native

### ⏱️ Cálculo de ETA
- Estimación basada en distancia (km)
- Velocidad promedio aproximada (50 km/h)
- Actualización en tiempo real

### 📍 Soporte Multiplataforma
- Android (vía Gradle)
- iOS (vía CocoaPods)
- Web (React)

## 📁 Estructura del Proyecto

```
proyecto-albergue/
├── README.md                          # Este archivo
├── DOCUMENTATION.md                   # Documentación general
├── TECHNICAL_DOCUMENTATION.md         # Documentación técnica
├── ARCHITECTURE.md                    # Arquitectura del sistema
├── MQTT_PROTOCOL.md                   # Protocolo MQTT
├── API_REFERENCE.md                   # Referencia de APIs
├── INSTALLATION_GUIDE.md              # Guía de instalación detallada
├── TROUBLESHOOTING.md                 # Guía de resolución de problemas
├── DIAGRAMS.md                        # Diagramas del proyecto
├── app-albergue/                      # App para gestores de albergue
│   ├── App.js                         # Componente principal
│   ├── app.json                       # Configuración Expo
│   ├── index.js                       # Punto de entrada
│   ├── package.json                   # Dependencias
│   ├── android/                       # Configuración Android
│   ├── assets/                        # Recursos estáticos
│   └── ...
├── app-repartidor/                    # App para repartidores
│   ├── App.js                         # Componente principal
│   ├── app.json                       # Configuración Expo
│   ├── index.js                       # Punto de entrada
│   ├── package.json                   # Dependencias
│   ├── assets/                        # Recursos estáticos
│   └── ...
└── .expo/                             # Configuración Expo
```

## 🔧 Requisitos Previos

### Software Requerido
- **Node.js** >= 18.x
- **npm** >= 9.x o **yarn** >= 1.22.x
- **Expo CLI**: `npm install -g expo-cli`
- **Git** para control de versiones

### Configuración del Broker MQTT
- **Broker MQTT** (ej. Mosquitto)
- **WebSocket habilitado** en el broker (puerto 9001)
- IP del broker: `192.168.1.137` (editable en código)

### Dispositivos/Emuladores
- Emulador Android SDK (Android Studio) o dispositivo físico
- O simulador iOS (solo macOS) o dispositivo físico
- Navegador web moderno para versión web

## 📦 Instalación

### 1. Clonar el Repositorio
```bash
git clone <tu-repo-url>
cd proyecto-albergue
```

### 2. Instalar Dependencias

#### Para App de Albergue
```bash
cd app-albergue
npm install
# o
yarn install
```

#### Para App de Repartidor
```bash
cd ../app-repartidor
npm install
# o
yarn install
```

### 3. Configurar el Broker MQTT

Ver [INSTALLATION_GUIDE.md](./INSTALLATION_GUIDE.md) para instrucciones detalladas sobre cómo configurar Mosquitto.

## ⚙️ Configuración

### Variables de Configuración (en App.js)

```javascript
const BROKER = 'ws://192.168.1.137:9001';
const BROKER_HOST = '192.168.1.137';
const BROKER_PORT = 9001;
const BROKER_PATH = '/mqtt';
const TOPIC_POSICION = 'paquetes/posicion';
const TOPIC_RUTA = 'paquetes/ruta';
```

**Modificar según tu entorno:**
- `BROKER_HOST`: IP del servidor MQTT
- `BROKER_PORT`: Puerto del broker (típicamente 8883 SSL o 1883 no-SSL)
- `BROKER_PATH`: Ruta WebSocket (típicamente `/mqtt`)

## 🚀 Uso

### Ejecutar App de Albergue

```bash
cd app-albergue
npm start
# o directamente en Android
npm run android
```

### Ejecutar App de Repartidor

```bash
cd app-repartidor
npm start
# o directamente en Android
npm run android
```

### Flujo Típico de Uso

1. **Albergue**: Inicia app, selecciona su ubicación/albergue
2. **Repartidor**: Inicia app, ingresa su nombre, presiona "Iniciar Viaje"
3. **Sistema**: 
   - Repartidor comparte ubicación cada 5 segundos
   - Albergue recibe ubicación y calcula ruta
   - Repartidor recibe ruta y destino
   - ETA se actualiza en tiempo real

## 🛠️ Tecnologías

### Frontend
- **React Native** - Framework multiplataforma
- **Expo** - Plataforma de desarrollo
- **React** 19.1.0 - Librería UI

### Mapas y Geolocalización
- **Leaflet.js** - Librería de mapas
- **OpenStreetMap** - Datos cartográficos
- **expo-location** - API de localización nativa

### Comunicación
- **Paho MQTT** - Cliente MQTT para JavaScript
- **WebView** - Integración de mapas HTML

### Otros
- **Axios** - Cliente HTTP
- **React Native WebView** - Componente web

## 📚 Documentación

- **[DOCUMENTATION.md](./DOCUMENTATION.md)** - Guía general y conceptos
- **[TECHNICAL_DOCUMENTATION.md](./TECHNICAL_DOCUMENTATION.md)** - Referencia técnica detallada
- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Arquitectura del sistema
- **[MQTT_PROTOCOL.md](./MQTT_PROTOCOL.md)** - Protocolo y mensajes MQTT
- **[API_REFERENCE.md](./API_REFERENCE.md)** - Referencia de componentes y funciones
- **[INSTALLATION_GUIDE.md](./INSTALLATION_GUIDE.md)** - Guía paso a paso de instalación
- **[TROUBLESHOOTING.md](./TROUBLESHOOTING.md)** - Resolución de problemas comunes
- **[DIAGRAMS.md](./DIAGRAMS.md)** - Diagramas visuales del sistema

## 🤝 Contribución

Las contribuciones son bienvenidas. Por favor:

1. Fork el proyecto
2. Crea una rama para tu feature (`git checkout -b feature/AmazingFeature`)
3. Commit tus cambios (`git commit -m 'Add some AmazingFeature'`)
4. Push a la rama (`git push origin feature/AmazingFeature`)
5. Abre un Pull Request

## 📝 Licencia

Este proyecto está bajo licencia MIT. Ver archivo `LICENSE` para más detalles.

## 📞 Contacto

Para preguntas o problemas, por favor abre un issue en el repositorio.

---

**Última actualización**: Noviembre 2025
**Versión**: 1.0.0
