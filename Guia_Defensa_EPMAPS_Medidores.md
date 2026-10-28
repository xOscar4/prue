# Guía rápida de defensa técnica – EPMAPS Medidores

## 1. Resumen de la aplicación y su funcionamiento

EPMAPS Medidores es una app móvil desarrollada con Expo y React Native para que un técnico registre lecturas de medidores de agua en campo. La app tiene un flujo principal de login, ruta de medidores, detalle del equipo, formulario de lectura, fotos, borrador automático y cola de sincronización.

El proyecto usa:

- Expo Router para navegación y rutas
- React Native para la interfaz
- TanStack Query para consultar ruta y historial
- AsyncStorage para guardar sesión, borrador, outbox y caché local
- Axios para llamar a MockAPI
- NetInfo para detectar conexión
- Expo Camera + Expo FileSystem + Expo Image para fotos

La lógica de negocio está organizada por responsabilidades:

- Autenticación: [src/features/auth/auth-provider.tsx](src/features/auth/auth-provider.tsx), [src/services/auth.ts](src/services/auth.ts), [src/features/auth/session.ts](src/features/auth/session.ts)
- Consultas y caché: [src/services/queryClient.ts](src/services/queryClient.ts), [src/features/medidores/hooks.ts](src/features/medidores/hooks.ts), [src/features/lecturas/hooks.ts](src/features/lecturas/hooks.ts)
- Outbox y sincronización: [src/features/lecturas/outbox.ts](src/features/lecturas/outbox.ts), [src/features/lecturas/outbox-provider.tsx](src/features/lecturas/outbox-provider.tsx), [src/features/lecturas/sync.ts](src/features/lecturas/sync.ts)
- Formulario y validación: [src/features/lecturas/use-lectura-form.ts](src/features/lecturas/use-lectura-form.ts), [src/utils/validators.ts](src/utils/validators.ts), [src/features/lecturas/guardar-lectura.ts](src/features/lecturas/guardar-lectura.ts)
- Fotos: [src/features/lecturas/foto.ts](src/features/lecturas/foto.ts), [src/features/lecturas/components/camera-modal.tsx](src/features/lecturas/components/camera-modal.tsx), [src/features/lecturas/components/photo-picker.tsx](src/features/lecturas/components/photo-picker.tsx)

La app no guarda la clave; solo guarda el perfil del técnico. También guarda la caché de consultas y la cola “outbox” para seguir trabajando sin señal.

---

## 2. Estructura del proyecto y archivos más importantes

### Carpetas principales

- [src/app](src/app): vistas y rutas de Expo Router.
- [src/components](src/components): componentes reutilizables de la interfaz.
- [src/constants](src/constants): tema, colores e iconos.
- [src/features](src/features): autenticación, medidores, lecturas y lógica de negocio por dominio.
- [src/services](src/services): API, caché, autenticación, errores.
- [src/types](src/types): modelos TypeScript.
- [src/utils](src/utils): validaciones y helpers de formato.
- [docs](docs): ADR, guion de defensa y pruebas.

### Archivos clave

| Archivo                                                                                | Función                                                                          |
| -------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| [package.json](package.json)                                                           | Declara dependencias y scripts del proyecto.                                     |
| [src/app/\_layout.tsx](src/app/_layout.tsx)                                            | Configura el layout global, el proveedor de caché y el banner de sincronización. |
| [src/app/index.tsx](src/app/index.tsx)                                                 | Login y validación del técnico.                                                  |
| [src/features/auth/session.ts](src/features/auth/session.ts)                           | Guarda y lee la sesión local con AsyncStorage.                                   |
| [src/services/api.ts](src/services/api.ts)                                             | Realiza peticiones a MockAPI y guarda copias locales de medidores.               |
| [src/services/queryClient.ts](src/services/queryClient.ts)                             | Configura TanStack Query y la persistencia en AsyncStorage.                      |
| [src/features/medidores/hooks.ts](src/features/medidores/hooks.ts)                     | Consulta la ruta y el detalle de medidores con caché.                            |
| [src/features/lecturas/use-lectura-form.ts](src/features/lecturas/use-lectura-form.ts) | Maneja el formulario, el borrador y la validación local.                         |
| [src/features/lecturas/guardar-lectura.ts](src/features/lecturas/guardar-lectura.ts)   | Crea la lectura local y evita duplicados en fecha o mes.                         |
| [src/features/lecturas/outbox.ts](src/features/lecturas/outbox.ts)                     | Guarda las lecturas pendientes.                                                  |
| [src/features/lecturas/sync.ts](src/features/lecturas/sync.ts)                         | Envía la cola de lecturas pendientes.                                            |
| [src/features/lecturas/draft.ts](src/features/lecturas/draft.ts)                       | Guarda y recupera los borradores.                                                |
| [src/features/lecturas/draft-gate.tsx](src/features/lecturas/draft-gate.tsx)           | Muestra el modal para continuar o descartar el borrador.                         |
| [src/components/status-banner.tsx](src/components/status-banner.tsx)                   | Muestra la conexión y pendientes en toda la app.                                 |
| [src/utils/validators.ts](src/utils/validators.ts)                                     | Reglas del negocio: formato del código, consumo, anomalía, campos obligatorios.  |
| [src/features/lecturas/foto.ts](src/features/lecturas/foto.ts)                         | Crea copia de la foto en documentos del teléfono.                                |

---

## 3. Preguntas sobre las librerías y dónde se utilizan

### AsyncStorage

- ¿Qué es? Es almacenamiento local persistente del dispositivo.
- ¿Dónde lo usaste? En [src/features/auth/session.ts](src/features/auth/session.ts), [src/features/lecturas/draft.ts](src/features/lecturas/draft.ts), [src/features/lecturas/outbox.ts](src/features/lecturas/outbox.ts), [src/services/api.ts](src/services/api.ts), [src/services/queryClient.ts](src/services/queryClient.ts).
- ¿Qué guardas? La sesión, el borrador, el outbox, los medidores vistos y la caché de consultas.
- Código que debes mostrar: [src/features/auth/session.ts](src/features/auth/session.ts#L1-L22), [src/features/lecturas/outbox.ts](src/features/lecturas/outbox.ts#L1-L47), [src/services/queryClient.ts](src/services/queryClient.ts#L1-L42).

### TanStack Query

- ¿Para qué utilizaste TanStack Query? Para cargar datos de la ruta, el detalle y el historial con caché, revalidación y manejo de offline-first.
- ¿Dónde se usa? [src/features/medidores/hooks.ts](src/features/medidores/hooks.ts), [src/features/lecturas/hooks.ts](src/features/lecturas/hooks.ts), [src/app/\_layout.tsx](src/app/_layout.tsx#L24-L34).
- ¿Cómo guardas la caché? Con [src/services/queryClient.ts](src/services/queryClient.ts#L1-L42) y `createAsyncStoragePersister`.
- ¿Qué hace `PersistQueryClientProvider`? Sirve para hidratar la caché al abrir la app y persistir la caché durante la sesión.

### @tanstack/query-async-storage-persister

- ¿Qué hace? Guarda el estado de las queries en AsyncStorage para que queden disponibles sin conexión.
- ¿Dónde se importa? [src/services/queryClient.ts](src/services/queryClient.ts#L8-L42).
- ¿Qué parte mostrar? La configuración del persister y del cliente global para mostrar la estrategia de persistencia.

### Axios

- ¿Qué hace? Crea un cliente para comunicar la app con MockAPI.
- ¿Dónde se usa? [src/services/api.ts](src/services/api.ts#L1-L85) y [src/services/api.ts](src/services/api.ts#L117-L199).
- ¿Qué peticiones hace? GET y POST a `/medidores` y `/lecturas`.
- Debes mostrar: la base URL, el interceptor y el envío de pantallas.

### NetInfo

- ¿Para qué sirve? Detecta si hay conexión móvil o Wi‑Fi y cambia el estado online en toda la app.
- ¿Dónde se importa? [src/hooks/use-network-status.ts](src/hooks/use-network-status.ts#L1-L22).
- ¿Cómo detectas la conexión? Con `NetInfo.addEventListener` y `NetInfo.fetch()`.
- ¿Dónde se usa la detección? En [src/components/status-banner.tsx](src/components/status-banner.tsx#L9-L76) y [src/features/lecturas/outbox-provider.tsx](src/features/lecturas/outbox-provider.tsx#L36-L127).

### Expo Camera

- ¿Dónde se usa? [src/features/lecturas/components/camera-modal.tsx](src/features/lecturas/components/camera-modal.tsx#L1-L169).
- ¿Qué hace? Abre la cámara, pide permisos y captura la foto del medidor.
- ¿Qué parte mostrar? El modal con `CameraView`, `useCameraPermissions`, `takePictureAsync` y la lógica de permiso.

### Expo FileSystem

- ¿Dónde se usa? [src/features/lecturas/foto.ts](src/features/lecturas/foto.ts#L1-L28).
- ¿Qué hace? Copia la foto desde la caché temporal a la carpeta de documentos del teléfono para que siga disponible.

### Expo Image

- ¿Dónde se usa? [src/features/lecturas/components/photo-picker.tsx](src/features/lecturas/components/photo-picker.tsx#L1-L119).
- ¿Para qué? Para mostrar la vista previa de la foto tomada dentro de la app.

### DateTimePicker

- ¿Dónde se usa? [src/features/medidores/components/campo-fecha.tsx](src/features/medidores/components/campo-fecha.tsx#L1-L98).
- ¿Para qué sirve? Para elegir la fecha de la lectura o de creación del medidor.

### Expo Router

- ¿Dónde se usa? En [src/app/\_layout.tsx](src/app/_layout.tsx#L1-L83) y en todas las pantallas bajo [src/app](src/app).
- ¿Qué hace? Define navegación, rutas dinámicas como `/medidor/[codigo]` y `/lectura/[codigo]` y el stack principal.

### Expo Linear Gradient

- ¿Dónde se usa? En [src/app/\_layout.tsx](src/app/_layout.tsx#L25-L76), [src/components/status-banner.tsx](src/components/status-banner.tsx#L13-L76), [src/app/index.tsx](src/app/index.tsx).
- ¿Qué función tiene? Da el fondo visual de agua que caracteriza a la app y ayuda a la identidad visual.

### Dependencias que están instaladas pero no identifico uso directo

- `expo-device`, `expo-status-bar`, `expo-web-browser`, `expo-symbols`, `expo-glass-effect`, `expo-system-ui`, `react-native-worklets`, `react-native-reanimated`, `react-native-gesture-handler`, `react-native-safe-area-context` están instaladas porque forman parte del entorno Expo/React Native. La app sí usa `react-native-safe-area-context` y varias de estas librerías quedan activas por el stack del proyecto, pero no todas se usan en la lógica de negocio principal que debes explicar en la defensa.

---

## 4. Preguntas sobre la lógica de negocio

### 4.1 ¿Cómo se registra una lectura de un medidor?

- Lógica: [src/features/lecturas/use-lectura-form.ts](src/features/lecturas/use-lectura-form.ts#L19-L104), [src/features/lecturas/guardar-lectura.ts](src/features/lecturas/guardar-lectura.ts#L17-L123).
- Respuesta corta: El usuario llena el formulario, validamos los campos, calculamos consumo y anomalia, y luego se guarda en el outbox local con estado pendiente de sincronización.
- Importante: la app guarda primero localmente, no directamente en MockAPI.

### 4.2 ¿Cómo se calcula el consumo?

- Lógica: [src/utils/validators.ts](src/utils/validators.ts#L17-L34), [src/features/lecturas/guardar-lectura.ts](src/features/lecturas/guardar-lectura.ts#L33-L61).
- Respuesta: `consumo = lectura actual - última lectura`. Si el resultado es negativo, se normaliza a 0, y si es mayor que el triple del promedio, se marca como anómalo.

### 4.3 ¿Cómo se validan los campos obligatorios?

- Lógica: [src/utils/validators.ts](src/utils/validators.ts#L82-L144), [src/features/lecturas/use-lectura-form.ts](src/features/lecturas/use-lectura-form.ts#L117-L187).
- Respuesta: `validarFormulario` revisa estado, fecha, lectura, motivo, descripción, foto y confirmación. Si falta algo, devuelve errores y bloquea el guardado.

### 4.4 ¿Cómo se detecta una lectura menor que la anterior?

- Lógica: [src/utils/validators.ts](src/utils/validators.ts#L35-L56), [src/features/lecturas/guardar-lectura.ts](src/features/lecturas/guardar-lectura.ts#L33-L61).
- Respuesta: con `esLecturaMenor(actual, anterior)`. Si el valor es menor, se marca como lectura con posible cambio de equipo y exige confirmación del usuario.

### 4.5 ¿Cómo se evita registrar dos lecturas del mismo medidor en el mismo mes?

- Lógica: [src/features/lecturas/guardar-lectura.ts](src/features/lecturas/guardar-lectura.ts#L63-L120), [src/features/lecturas/hooks.ts](src/features/lecturas/hooks.ts#L8-L26), [src/services/api.ts](src/services/api.ts#L171-L222).
- Respuesta: la app busca si ya existe la misma lectura en el outbox o en el historial enviado, comparando medidor + fecha o periodo. El servidor también revisa `idempotencia` y `medidorCodigo + periodo` antes de insertar.
- Nota importante: la protección de duplicados es local y en la lógica de la app; la validación del servidor se limita a lo que está en este código y se valida antes del `POST`.

### 4.6 ¿Cómo se distingue una lectura pendiente de una enviada?

- Lógica: [src/features/lecturas/outbox.ts](src/features/lecturas/outbox.ts#L1-L47) y [src/features/lecturas/outbox-provider.tsx](src/features/lecturas/outbox-provider.tsx#L38-L128).
- Respuesta: las lecturas pendientes viven en el outbox con `estadoSync: 'pendiente'` o `estadoSync: 'enviando'`; las enviadas ya no están en el outbox y se cargan desde la API con `useHistorial`.

### 4.7 ¿Si una lectura pendiente se puede corregir y qué pasa con una enviada?

- Lógica: [src/features/lecturas/outbox-provider.tsx](src/features/lecturas/outbox-provider.tsx#L38-L128), [src/app/(tabs)/historial.tsx](<src/app/(tabs)/historial.tsx#L16-L76>), [src/features/lecturas/guardar-lectura.ts](src/features/lecturas/guardar-lectura.ts#L90-L123).
- Respuesta: una lectura pendiente puede abrirse para editarse antes de enviarse. Las enviadas son de solo lectura y no tienen acción de edición.

### 4.8 ¿Cómo funciona el modo sin conexión?

- Lógica: [src/hooks/use-network-status.ts](src/hooks/use-network-status.ts#L1-L22), [src/features/lecturas/outbox-provider.tsx](src/features/lecturas/outbox-provider.tsx#L45-L127), [src/components/status-banner.tsx](src/components/status-banner.tsx#L11-L76).
- Respuesta: si pierde conexión, la app sigue funcionando con caché local, guarda la lectura en el outbox y reintenta el envío cuando vuelve la conexión.

### 4.9 ¿Cómo se guardan y sincronizan las lecturas pendientes?

- Lógica: [src/features/lecturas/outbox.ts](src/features/lecturas/outbox.ts#L1-L47), [src/features/lecturas/sync.ts](src/features/lecturas/sync.ts#L1-L52), [src/features/lecturas/outbox-provider.tsx](src/features/lecturas/outbox-provider.tsx#L47-L127).
- Respuesta: se guardan en AsyncStorage y se envían secuencialmente. Si falla el envío, se dejan en estado pendiente o error y se reintentan.

### 4.10 ¿Qué pasa si falla la sincronización?

- Lógica: [src/features/lecturas/sync.ts](src/features/lecturas/sync.ts#L8-L52), [src/features/lecturas/outbox-provider.tsx](src/features/lecturas/outbox-provider.tsx#L55-L79).
- Respuesta: la lectura no desaparece del outbox; queda con error o pendiente y el flujo muestra mensaje en el banner. Si la red falla, la cola se reintenta más tarde.

### 4.11 ¿Cómo se gestionan las fotografías?

- Lógica: [src/features/lecturas/components/camera-modal.tsx](src/features/lecturas/components/camera-modal.tsx#L15-L146), [src/features/lecturas/foto.ts](src/features/lecturas/foto.ts#L1-L28), [src/features/lecturas/components/photo-picker.tsx](src/features/lecturas/components/photo-picker.tsx#L13-L106).
- Respuesta: la foto se toma de la cámara, se guarda en una carpeta del sistema de archivos del teléfono y luego se adjunta a la lectura. Si es obligatoria, no deja guardar sin ella.

### 4.12 ¿Cómo se conserva la sesión del usuario?

- Lógica: [src/features/auth/session.ts](src/features/auth/session.ts#L1-L22), [src/features/auth/auth-provider.tsx](src/features/auth/auth-provider.tsx#L1-L45), [src/app/index.tsx](src/app/index.tsx#L15-L84).
- Respuesta: al iniciar sesión, se guarda el técnico en AsyncStorage. Al abrir la app, se lee la sesión y, si existe, se redirige directamente a la ruta.

### 4.13 ¿Qué datos se guardan localmente y cuáles se envían al servidor?

- Local: sesión, borrador, outbox, caché Query, fotos en documentos, medidores vistos.
- Servidor: medidores y lecturas en MockAPI.
- Lógica: [src/services/api.ts](src/services/api.ts#L1-L222), [src/services/queryClient.ts](src/services/queryClient.ts#L1-L42), [src/features/auth/session.ts](src/features/auth/session.ts#L1-L22), [src/features/lecturas/draft.ts](src/features/lecturas/draft.ts#L1-L29), [src/features/lecturas/outbox.ts](src/features/lecturas/outbox.ts#L1-L47).
- Limitación real: las fotos se guardan localmente en el teléfono y MockAPI recibe la URI local, no un archivo binario.

---

## 5. Fragmentos de código que debes aprender a explicar

### Fragmento 1 — Persistencia de consulta y caché

- Pregunta del ingeniero: “¿Por qué la app sigue funcionando sin internet y cómo se guarda la caché?”
- Archivo exacto: [src/services/queryClient.ts](src/services/queryClient.ts#L1-L42)
- Código importante:

```ts
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,
      gcTime: UNA_SEMANA,
      retry: 2,
      networkMode: "offlineFirst",
      refetchOnReconnect: true,
    },
  },
});

export const asyncStoragePersister = createAsyncStoragePersister({
  storage: AsyncStorage,
  key: "EPMAPS_QUERY_CACHE",
  throttleTime: 1000,
});
```

- ¿Qué hace? Configura la caché global de React Query para guardar consultas y reutilizarlas sin conexión.
- Cómo lo explico: “La app usa TanStack Query con persistencia local; si hay internet se refresca, y si no, toma la caché guardada en AsyncStorage.”
- Si no existiera: la ruta y el detalle del medidor no estarían disponibles sin conexión.

### Fragmento 2 — Sesión persistida

- Pregunta del ingeniero: “¿Dónde guardas la sesión y cómo se mantiene al cerrar la app?”
- Archivo exacto: [src/features/auth/session.ts](src/features/auth/session.ts#L1-L22)
- Código importante:

```ts
const KEY = "EPMAPS_SESION";

export async function leerSesion(): Promise<Tecnico | null> {
  const raw = await AsyncStorage.getItem(KEY);
  return raw ? (JSON.parse(raw) as Tecnico) : null;
}

export async function guardarSesion(tecnico: Tecnico): Promise<void> {
  await AsyncStorage.setItem(KEY, JSON.stringify(tecnico));
}
```

- ¿Qué hace? Guarda el perfil del técnico sin la clave y lo recupera al abrir la app.
- Cómo lo explico: “La sesión se guarda en AsyncStorage y se lee al arrancar; si existe, la app entra directamente a la ruta.”
- Si no existiera: cada vez que se cerrara la app, el usuario tendría que volver a iniciar sesión.

### Fragmento 3 — Outbox y sincronización

- Pregunta del ingeniero: “¿Cómo guardas las lecturas pendientes y cómo las envías?”
- Archivo exacto: [src/features/lecturas/outbox.ts](src/features/lecturas/outbox.ts#L1-L47), [src/features/lecturas/sync.ts](src/features/lecturas/sync.ts#L1-L52)
- Código importante:

```ts
export async function guardarOutbox(lecturas: LecturaLocal[]): Promise<void> {
  await AsyncStorage.setItem(KEY, JSON.stringify(lecturas));
}

export async function sincronizarCola(): Promise<ResultadoSync> {
  let items = await leerOutbox();
  for (const pendiente of pendientes) {
    try {
      await enviarLectura(pendiente);
      items = items.filter((item) => item.id !== pendiente.id);
      await guardarOutbox(items);
    } catch {
      // se reintenta más tarde
    }
  }
}
```

- ¿Qué hace? La app guarda la lectura localmente y, cuando vuelve la conexión, la intenta enviar en orden.
- Cómo lo explico: “La lectura no se pierde; vive en el outbox hasta que el servidor confirma su recepción.”
- Si no existiera: si se perdiera la conexión, el registro se perdería o quedaría incompleto.

### Fragmento 4 — Validación de lectura y reglas del negocio

- Pregunta del ingeniero: “¿Cómo impediste que se registren lecturas inválidas o duplicadas?”
- Archivo exacto: [src/utils/validators.ts](src/utils/validators.ts#L1-L144), [src/features/lecturas/guardar-lectura.ts](src/features/lecturas/guardar-lectura.ts#L17-L123)
- Código importante:

```ts
if (form.estado === "normal" && lectura === null) {
  return {
    ok: false,
    mensaje: "La lectura es obligatoria para un medidor normal.",
  };
}

const repetida =
  lecturas.find(
    (item) =>
      item.medidorCodigo === codigo &&
      (item.fecha === fechaLectura || item.periodo === periodo),
  ) ??
  historial.find(
    (item) =>
      item.medidorCodigo === codigo &&
      (item.fecha === fechaLectura || item.periodo === periodo),
  );
```

- ¿Qué hace? Verifica que el medidor tenga lectura válida, que no haya otra lectura en la misma fecha o mes y que el flujo de validación cumpla las reglas del negocio.
- Cómo lo explico: “La app valida antes de guardar; así se evita que el técnico registre un dato inválido o duplicado.”
- Si no existiera: el usuario podría registrar errores, duplicados y lecturas no consistentes.

### Fragmento 5 — Borrador y recuperación de formulario

- Pregunta del ingeniero: “¿Qué pasa si el técnico cierra la app antes de terminar la lectura?”
- Archivo exacto: [src/features/lecturas/use-lectura-form.ts](src/features/lecturas/use-lectura-form.ts#L52-L146), [src/features/lecturas/draft-gate.tsx](src/features/lecturas/draft-gate.tsx#L11-L69)
- Código importante:

```ts
useEffect(() => {
  if (guardadoOk.current || existente || !hayDatos(form)) return;
  const timer = setTimeout(() => {
    void guardarBorrador({
      codigo,
      ...form,
      actualizadoEn: new Date().toISOString(),
    });
  }, 250);
  return () => clearTimeout(timer);
}, [form, codigo, cargando, existente, borrador]);
```

- ¿Qué hace? Guarda el formulario en AsyncStorage mientras se está llenando y ofrece continuar o descartar cuando se abre la app.
- Cómo lo explico: “El borrador es un respaldo, como un salvavidas del formulario; si la app se cierra, no se pierde el trabajo.”
- Si no existiera: la pérdida de un registro incompleto sería muy frecuente y frustrante para el técnico.

### Fragmento 6 — Fotos y almacenamiento en documentos

- Pregunta del ingeniero: “¿Cómo almacenas la foto del medidor y por qué la guardas en documentos?”
- Archivo exacto: [src/features/lecturas/foto.ts](src/features/lecturas/foto.ts#L1-L28), [src/features/lecturas/components/camera-modal.tsx](src/features/lecturas/components/camera-modal.tsx#L14-L61)
- Código importante:

```ts
const carpeta = new Directory(Paths.document, CARPETA);
if (!carpeta.exists) {
  carpeta.create({ intermediates: true, idempotent: true });
}
const destino = new File(carpeta, `${nombre}.jpg`);
await new File(uri).copy(destino, { overwrite: true });
return destino.uri;
```

- ¿Qué hace? Copia la foto desde la caché temporal del sistema a una ubicación más permanente del teléfono.
- Cómo lo explico: “La foto no queda solo en la caché de la cámara; se guarda en documentos para que siga disponible aunque se cierre la app o se reinicie el teléfono.”
- Si no existiera: la imagen podría desaparecer cuando se limpia la caché o se reinicia el sistema.

---

## 6. Preguntas difíciles que podría hacer el ingeniero

1. “¿Qué parte del sistema realmente protege contra lecturas duplicadas?”
   - Respuesta: la app valida en la interfaz y en la lógica antes de guardar, y también revisa idempotencia y periodo antes del POST. Eso evita duplicados en el flujo principal, pero la firma de protección real del backend depende del código del servidor que no está aquí.

2. “Si la app se cierra justo cuando está enviando, ¿qué pasa?”
   - Respuesta: la lectura sigue en el outbox y se reintenta al recuperar la conexión; la idempotencia ayuda a evitar duplicados si la petición llegó pero no se recibió la respuesta.

3. “¿Qué pasa si hay una lectura con consumo anómalo?”
   - Respuesta: la app exige confirmación y, si el caso es dañado, exige foto y descripción. Eso está en [src/utils/validators.ts](src/utils/validators.ts#L82-L144).

4. “¿Dónde se valida la sesión?”
   - Respuesta: en [src/features/auth/auth-provider.tsx](src/features/auth/auth-provider.tsx#L1-L45) y [src/services/auth.ts](src/services/auth.ts#L1-L25).

5. “¿Qué pasa si el medidor tiene lectura menor a la anterior?”
   - Respuesta: se detecta con `esLecturaMenor` y exige confirmación, porque puede significar cambio de equipo.

6. “¿Qué harías si te preguntan por la diferencia entre caché de Query y outbox?”
   - Respuesta: la caché es para consultas y datos de lectura de la ruta; el outbox es la cola de lecturas pendientes que aún no se confirmaron en el servidor.

---

## 7. Orden recomendado para demostrar la aplicación en vivo

1. Login: entrar con `EMP-001` y `epmaps2026`.
2. Mostrar la ruta y un detalle de medidor.
3. Mostrar que la consulta usa caché y que al volver sin conexión se siguen leyendo los datos.
4. Registrar una lectura normal y mostrar el outbox o el estado “pendiente”.
5. Probar una lectura anómala con consumo triple del promedio y exigir confirmación.
6. Probar un medidor dañado con foto obligatoria.
7. Cerrar la app y volver a abrirla para mostrar el borrador.
8. Activar modo avión y enviar la cola de pendientes desde el historial o desde el banner.
9. Explicar qué ocurre si falla la red y qué componentes se usan para sincronizar.

---

## 8. Qué archivos debo abrir en VS Code durante la defensa

### 1) [src/app/index.tsx](src/app/index.tsx)

- Qué mostrar: flujo completo de login.
- Líneas sugeridas: [src/app/index.tsx](src/app/index.tsx#L15-L84)
- Pregunta del ingeniero: “¿Cómo validas el login y cómo mantienes la sesión?”
- Respuesta breve: “El login compara usuario y clave en la app y guarda el perfil del técnico en AsyncStorage. Si ya hay sesión, la app redirige a la ruta.”
- Demostración: entrar con las credenciales de prueba y volver a abrir la app.

### 2) [src/features/auth/session.ts](src/features/auth/session.ts#L1-L22)

- Qué mostrar: guardar y leer la sesión.
- Pregunta: “¿Dónde guardas y recuperas la sesión?”
- Respuesta breve: “En AsyncStorage con la clave `EPMAPS_SESION`.”
- Demostración: abrir la app, cerrar sesión y volver a entrar.

### 3) [src/features/medidores/hooks.ts](src/features/medidores/hooks.ts#L1-L43)

- Qué mostrar: consulta de ruta y detalle con `useQuery`.
- Pregunta: “¿Cómo consultas la ruta del día y cómo funciona la caché?”
- Respuesta breve: “Usamos TanStack Query con `staleTime` y persistencia en AsyncStorage; si falla la red, la app devuelve la información guardada.”
- Demostración: buscar medidor, guardar un código válido, cerrar internet y volver a abrir el detalle.

### 4) [src/utils/validators.ts](src/utils/validators.ts#L1-L144)

- Qué mostrar: validación del formulario.
- Pregunta: “¿Cómo evitas que el usuario registre datos inválidos?”
- Respuesta breve: “Todo se valida de forma local antes de guardar: código, fecha, lectura, campos del estado, foto y confirmación en casos anómalos.”
- Demostración: intentar guardar un medidor normal sin lectura, un dañado sin descripción o un consumo anómalo sin confirmar.

### 5) [src/features/lecturas/guardar-lectura.ts](src/features/lecturas/guardar-lectura.ts#L17-L123)

- Qué mostrar: creación de la lectura y bloqueo de duplicados.
- Pregunta: “Muéstrame el código que impide registrar dos lecturas del mismo medidor en un mes.”
- Respuesta breve: “La app busca una lectura previa en el outbox y en el historial, y compara medidor + fecha o periodo. Si ya existe, niega el guardado.”
- Demostración: intentar registrar dos veces el mismo medidor del mismo mes.

### 6) [src/features/lecturas/outbox.ts](src/features/lecturas/outbox.ts#L1-L47)

- Qué mostrar: colección de lecturas pendientes.
- Pregunta: “¿Cómo guardas las lecturas pendientes?”
- Respuesta breve: “Las pendientes se guardan en AsyncStorage en una lista llamada outbox.”
- Demostración: guardar una lectura con modo avión y verla en historial.

### 7) [src/features/lecturas/sync.ts](src/features/lecturas/sync.ts#L1-L52)

- Qué mostrar: envío de la cola y reintentos.
- Pregunta: “¿Qué pasa si falla la sincronización?”
- Respuesta breve: “La lectura se queda en el outbox con estado pendiente o error; se reintenta cuando haya red.”
- Demostración: activar modo avión, guardar varias lecturas y volver a la conexión.

### 8) [src/features/lecturas/foto.ts](src/features/lecturas/foto.ts#L1-L28)

- Qué mostrar: almacenamiento local de la foto.
- Pregunta: “¿Cómo guardas la fotografía?”
- Respuesta breve: “La foto se copia de la caché a la carpeta `fotos-medidores` dentro de documentos del teléfono.”
- Demostración: tomar una foto, cerrar la app y volver a entrar al formulario.

### 9) [src/features/lecturas/draft.ts](src/features/lecturas/draft.ts#L1-L29)

- Qué mostrar: borrador y recuperación.
- Pregunta: “¿Qué pasa si el usuario abandona una lectura incompleta?”
- Respuesta breve: “El borrador se guarda en AsyncStorage; luego el modal de continuación le permite seguir o descartar.”
- Demostración: empezar una lectura, salir a la app principal y volver a la ruta.

### 10) [src/components/status-banner.tsx](src/components/status-banner.tsx#L9-L76)

- Qué mostrar: banner global de conexión y pendientes.
- Pregunta: “¿Cómo sabes si hay conexión y cuántas lecturas faltan por enviar?”
- Respuesta breve: “NetInfo indica el estado y el banner muestra cuántas lecturas hay pendientes o con error.”
- Demostración: ver el banner al entrar sin conexión y después al recuperar la red.

---

## 9. Resumen para la defensa

Si te hacen una pregunta del tipo “¿Dónde utilizaste AsyncStorage?”, la respuesta correcta es: “En la sesión, en el outbox, en el borrador, en la caché de consultas y en los medidores vistos”.

Si te hacen “Muéstrame el código que impide registrar dos lecturas del mismo medidor en un mes”, debes abrir [src/features/lecturas/guardar-lectura.ts](src/features/lecturas/guardar-lectura.ts#L63-L120) y explicar la comprobación de `medidorCodigo` + `periodo` y la búsqueda en el historial.

Si te hacen “¿Cómo funciona esa parte?”, la clave está en responder con claridad y sin memorizar frases largas: primero explicas qué hace, luego qué datos usa, y luego por qué es necesario para la experiencia del técnico en campo.

No inventes funciones ni librerías. Todo lo anterior se basa en el código real del proyecto abierto en VS Code y en las dependencias declaradas en [package.json](package.json).
