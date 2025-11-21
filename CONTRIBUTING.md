# 🤝 Guía de Contribución - Proyecto Albergue

Instrucciones para contribuir al proyecto y mantener la calidad del código.

## 📋 Tabla de Contenidos

1. [Código de Conducta](#código-de-conducta)
2. [Cómo Contribuir](#cómo-contribuir)
3. [Proceso de Pull Request](#proceso-de-pull-request)
4. [Estándares de Código](#estándares-de-código)
5. [Documentación](#documentación)
6. [Testing](#testing)
7. [Commit Messages](#commit-messages)
8. [Issues y Bugs](#issues-y-bugs)

## 📜 Código de Conducta

Todos los contribuyentes deben respetar este código de conducta:

- ✅ Ser respetuoso con otros miembros del equipo
- ✅ Brindar retroalimentación constructiva
- ✅ Aceptar críticas con profesionalismo
- ✅ Mantener los espacios inclusivos
- ❌ No tolerancia con acoso, discriminación u hostilidad

## 🚀 Cómo Contribuir

### Paso 1: Fork y Clone

```bash
# Fork en GitHub
# Clonar tu fork
git clone https://github.com/tu-usuario/proyecto-albergue.git
cd proyecto-albergue

# Agregar upstream
git remote add upstream https://github.com/original-repo/proyecto-albergue.git
```

### Paso 2: Crear Rama

```bash
# Actualizar main
git checkout main
git pull upstream main

# Crear rama para tu feature
git checkout -b feature/nombre-descriptivo

# O para bugfix
git checkout -b bugfix/nombre-del-bug
```

**Convención de nombres**:
- `feature/añadir-autenticacion`
- `bugfix/conexion-mqtt-timeout`
- `docs/actualizar-readme`
- `refactor/mejorar-calculo-eta`

### Paso 3: Hacer Cambios

```bash
# Editar archivos
# Hacer commits (ver sección Commit Messages)

# Verificar cambios
git status
git diff

# Agregar cambios
git add archivo.js
# O todos los cambios
git add .

# Commit
git commit -m "feat: descripción del cambio"
```

### Paso 4: Push a tu Fork

```bash
git push origin feature/nombre-descriptivo
```

## 🔄 Proceso de Pull Request

### Crear PR

1. Ve a GitHub y crea un Pull Request
2. Completa la plantilla PR (si existe)
3. Referencia issues relacionados: `Fixes #123` o `Related to #456`

### Plantilla PR

```markdown
## Descripción
Breve descripción de los cambios

## Tipo de Cambio
- [ ] Bugfix
- [ ] Feature
- [ ] Breaking change
- [ ] Documentación

## Cambios Realizados
- Cambio 1
- Cambio 2
- Cambio 3

## Testing
- [ ] Probado localmente
- [ ] Probado en dispositivo
- [ ] Pruebas unitarias pasadas

## Checklist
- [ ] Mi código sigue los estándares
- [ ] He actualizado la documentación
- [ ] No hay conflictos con main
- [ ] Los commits tienen mensajes claros
```

### Revisión

- Espera revisión de al menos un mantenedor
- Responde comentarios de revisión
- Realiza cambios solicitados
- Solicita re-revisión cuando esté listo

## 📝 Estándares de Código

### JavaScript/React

```javascript
// ✅ BIEN
const calcularETA = async (origen, destino) => {
  const distancia = calcularDistancia(origen, destino);
  const minutos = Math.ceil(distancia / 0.8);
  return minutos;
};

// ❌ MAL
const CalcularETA = async (o, d) => {
  let dist = (o.lat - d.lat) ** 2 + (o.lng - d.lng) ** 2;
  return Math.ceil(Math.sqrt(dist) * 111 / 0.8);
};
```

### Convenciones

| Aspecto | Estándar |
|---------|----------|
| **Indentación** | 2 espacios (no tabs) |
| **Punto y coma** | Obligatorio |
| **Comillas** | Comillas simples para strings |
| **Variables** | camelCase |
| **Constantes** | UPPER_SNAKE_CASE |
| **Funciones** | camelCase |
| **Clases** | PascalCase |
| **Componentes React** | PascalCase |

### Ejemplo Completo

```javascript
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet } from 'react-native';

// Constantes
const BROKER_HOST = '192.168.1.137';
const UPDATE_INTERVAL = 5000; // ms

// Componente
export default function App() {
  // Estado
  const [coords, setCoords] = useState(null);
  const [isLoading, setIsLoading] = useState(false);

  // Efectos
  useEffect(() => {
    initializeApp();
  }, []);

  // Funciones
  const initializeApp = async () => {
    try {
      setIsLoading(true);
      await setupMQTT();
      await requestLocationPermission();
    } catch (error) {
      console.error('Error inicializando:', error);
    } finally {
      setIsLoading(false);
    }
  };

  const setupMQTT = async () => {
    // Implementación
  };

  const requestLocationPermission = async () => {
    // Implementación
  };

  // Render
  return (
    <View style={styles.container}>
      <Text>App Albergue</Text>
    </View>
  );
}

// Estilos
const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 16,
  },
});
```

### ESLint y Prettier (Opcional)

```bash
# Instalar
npm install --save-dev eslint prettier eslint-config-prettier

# Configurar .eslintrc.json
{
  "extends": ["airbnb-base", "prettier"],
  "env": {
    "browser": true,
    "node": true,
    "es2021": true
  }
}

# Ejecutar
npm run lint       # Revisar
npm run lint:fix   # Arreglar
```

## 📚 Documentación

### Comentarios en Código

```javascript
// ✅ BIEN: Comentario descriptivo
// Calcula la distancia euclidiana en km entre dos puntos
const calcularDistancia = (p1, p2) => {
  const latDiff = p2.lat - p1.lat;
  const lngDiff = p2.lng - p1.lng;
  return Math.sqrt(latDiff ** 2 + lngDiff ** 2) * 111;
};

// ❌ MAL: Comentario obvio
// Calcular distancia
const d = (p1, p2) => { /* ... */ };
```

### JSDoc para Funciones Complejas

```javascript
/**
 * Calcula la ruta entre dos puntos interpolando coordenadas intermedias.
 *
 * @param {Object} origen - Punto de inicio {lat, lng}
 * @param {Object} destino - Punto final {lat, lng}
 * @param {number} numPuntos - Número de puntos interpolados (default: 20)
 * @returns {Array} Array de coordenadas [[lat, lng], ...]
 * @throws {Error} Si coordenadas son inválidas
 *
 * @example
 * const ruta = calcularRuta(
 *   {lat: 20.375, lng: -99.982},
 *   {lat: 20.389, lng: -99.997}
 * );
 * // Retorna: [[20.375, -99.982], [20.376, ...], ...]
 */
const calcularRuta = (origen, destino, numPuntos = 20) => {
  // Implementación
};
```

### Actualizar Documentación

Si cambias funcionalidad:
1. Actualiza TECHNICAL_DOCUMENTATION.md
2. Actualiza API_REFERENCE.md si aplica
3. Actualiza ARCHITECTURE.md si es cambio mayor
4. Actualiza README.md si afecta instalación/uso

## 🧪 Testing

### Unit Tests (Recomendado)

```javascript
// __tests__/calculos.test.js
import { calcularETA, calcularDistancia } from '../calculos';

describe('Cálculos', () => {
  describe('calcularDistancia', () => {
    it('debe calcular distancia correctamente', () => {
      const resultado = calcularDistancia(
        { lat: 0, lng: 0 },
        { lat: 0, lng: 0.009 }
      );
      expect(resultado).toBeCloseTo(1, 1); // ~1 km
    });

    it('debe manejar puntos idénticos', () => {
      const resultado = calcularDistancia(
        { lat: 20.375, lng: -99.982 },
        { lat: 20.375, lng: -99.982 }
      );
      expect(resultado).toBe(0);
    });
  });

  describe('calcularETA', () => {
    it('debe calcular ETA en minutos', () => {
      const resultado = calcularETA(
        { lat: 0, lng: 0 },
        { lat: 0, lng: 0.4 }
      );
      expect(resultado).toBeGreaterThan(0);
      expect(resultado).toBeLessThan(100);
    });
  });
});
```

### Testing Manual

Antes de PR:
- [ ] Probado en emulador Android
- [ ] Probado en dispositivo físico si es posible
- [ ] Probado menú principal
- [ ] Probado conexión MQTT
- [ ] Probado visualización de mapas
- [ ] Probado con repartidor real

## 💬 Commit Messages

### Formato Convencional

```
<tipo>(<alcance>): <descripción>

<cuerpo>

<footer>
```

### Tipos

- **feat**: Nueva característica
- **fix**: Corrección de bug
- **docs**: Cambios en documentación
- **style**: Cambios de formato (no código)
- **refactor**: Refactorización de código
- **perf**: Mejoras de rendimiento
- **test**: Agregar/modificar tests
- **chore**: Cambios en build, deps, etc.

### Ejemplos

```bash
# Feature simple
git commit -m "feat: agregar validación de coordenadas"

# Con alcance
git commit -m "feat(mqtt): agregar reconexión automática"

# Con descripción detallada
git commit -m "fix(location): resolver problema de GPS no actualiza

- Aumentar timeInterval a 3 segundos
- Validar precisión antes de usar
- Agregar logs de debug"

# Breaking change
git commit -m "feat!: cambiar estructura de mensaje MQTT

BREAKING CHANGE: formato de paquetes/posicion ha cambiado
Mirar MIGRATION.md para detalles"
```

## 🐛 Issues y Bugs

### Reportar un Bug

**Usar plantilla**:
```markdown
## Descripción del Bug
Descripción clara y concisa

## Pasos para Reproducir
1. ...
2. ...
3. ...

## Comportamiento Esperado
Qué debería pasar

## Comportamiento Actual
Qué pasa realmente

## Screenshots/Logs
Si aplica, adjuntar

## Ambiente
- OS: [Windows/macOS/Linux]
- Dispositivo: [Emulador/Físico]
- App: [Albergue/Repartidor]
- Versión: [ej: 1.0.0]
```

### Sugerir Feature

**Usar plantilla**:
```markdown
## Descripción
Descripción clara de la característica

## Caso de Uso
Por qué es necesaria

## Solución Propuesta
Cómo implementarla

## Alternativas Consideradas
Otras opciones

## Contexto Adicional
Links, referencias, etc.
```

## 📊 Proceso de Release

### Versionado Semántico

Versión: MAJOR.MINOR.PATCH (ej: 1.2.3)

- **MAJOR**: Cambios incompatibles
- **MINOR**: Nuevas características compatibles
- **PATCH**: Correcciones de bugs

### Pasos Release

1. Crear rama `release/v1.2.0`
2. Actualizar versión en package.json
3. Actualizar CHANGELOG.md
4. Merge a main y tag
5. Merge a develop

## 📞 Contacto

- **Issues**: Usar GitHub Issues
- **Discussiones**: Usar GitHub Discussions
- **Email**: [si es aplicable]
- **Slack/Discord**: [si es aplicable]

---

**Última actualización**: Noviembre 2025
**Versión**: 1.0.0

