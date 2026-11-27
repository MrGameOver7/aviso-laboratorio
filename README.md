# aviso-laboratorio

Prácticas de laboratorio del curso **Programación de Aplicaciones Móviles** (EPISIS | FISME | UNTRM).

**Autor:** Rony Marines Quincho

Aplicación móvil hecha con **React Native + Expo (SDK 58) y TypeScript** en la que se trabajaron dos prácticas sobre el mismo proyecto:

1. **Notificaciones locales**: la app genera un aviso en el propio teléfono.
2. **Notificaciones push remotas**: un servidor (Cloudflare Worker) genera el mensaje y lo entrega mediante Expo Push Service y Firebase Cloud Messaging (FCM).

---

## Práctica 1: Notificaciones locales

Al pulsar **"Enviar notificación"**, la app pide el permiso de notificaciones, crea el canal Android `laboratorio` y programa un aviso que aparece a los 3 segundos en la bandeja del teléfono.

- **Título:** `Hola mundo`
- **Mensaje:** `Prueba programada el <fecha y hora>`. La fecha se genera en el teléfono con `new Date().toLocaleString('es-PE')`.

Es una notificación **local** porque la crea el propio dispositivo a pedido de la app, sin servidor ni internet.

## Práctica 2: Notificaciones push con Cloudflare y Firebase

Al pulsar **"Pedir push al servidor"**:

1. La app obtiene su **ExpoPushToken** (`getExpoPushToken`).
2. Envía **solo ese token** al Cloudflare Worker (`POST /send`).
3. El Worker genera la **hora de Lima** y un **código aleatorio**, y le pide a Expo que envíe el aviso.
4. Expo entrega el push al teléfono mediante **FCM** (Firebase).
5. La app muestra el ticket con el que Expo aceptó el envío.

```
App  →  Cloudflare Worker  →  Expo Push Service  →  FCM  →  Teléfono Android
```

En la bandeja aparece **"Aviso desde el servidor"**, con la hora del servidor y el código. La hora y el código se generan en Cloudflare, no en el teléfono.

---

## Estructura

```
aviso-laboratorio/
├─ App.tsx                              # Pantalla con los dos botones
├─ app.json                             # Configuración de Expo (package Android, plugin, projectId)
├─ eas.json                             # Perfiles de compilación EAS (development)
├─ google-services.json                 # Configuración de la app Android en Firebase
└─ src/notifications/
   ├─ localNotifications.ts             # Handler, canal, permiso, aviso local y token push
   └─ pushServer.ts                     # Petición HTTP al Cloudflare Worker
```

| Archivo | Qué hace |
|---|---|
| `localNotifications.ts` | Configura el handler, crea el canal `laboratorio`, verifica el permiso (`ensureNotificationPermission`), programa el aviso local (`showLocalNotification`) y obtiene el token (`getExpoPushToken`). |
| `pushServer.ts` | Envía el token al Worker (`requestPushFromServer`) y devuelve el ticket de Expo. |
| `App.tsx` | Vincula los botones con sus acciones (`handleNotify` y `handleServerPush`) y muestra el estado de la solicitud remota. |

## Tecnologías

- React Native 0.88 + Expo SDK 58 + TypeScript
- `expo-notifications`, `expo-constants`, `expo-dev-client`
- EAS Build (development build para Android)
- Firebase Cloud Messaging (FCM V1)
- Cloudflare Workers (servidor de la práctica, preparado por el docente)

## Cómo ejecutar

```bash
npm install
npx expo start --dev-client --clear
```

- **Práctica 1:** funciona en **Expo Go** (`npx expo start`).
- **Práctica 2:** necesita la **development build** (`eas build --platform android --profile development`), porque Expo Go ya no admite push remoto en Android.

## Notas

- En Expo Go (SDK 58), crear el canal Android puede fallar con un error nativo. Por eso la creación del canal está dentro de un `try/catch`; si falla, Android usa el canal por defecto y el aviso igual aparece.
- `google-services.json` solo contiene identificadores de configuración. La **clave privada** de la cuenta de servicio de Firebase **no** está en el repositorio: se subió directamente a las credenciales de Expo.
