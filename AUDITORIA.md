# Auditoría del sistema — Catálogo de Proveedores CANIRAC Laguna

Fecha: 20 de octubre de 2026 · Alcance: `index.html`, `app.js`, `admin.html`, `store.js`, `ui.js`, `styles.css`,
`sw.js`, `pwa.js`, manifiestos, `firestore.rules`, `storage.rules`, `config.js`, `README.md`.

Leyenda de severidad: 🔴 Crítica · 🟠 Alta · 🟡 Media · 🟢 Baja

---

## 1. Acciones que debes hacer tú (no se pueden hacer desde el código)

| # | Acción | Por qué |
|---|--------|---------|
| 1 | 🔴 **Cambiar la contraseña del administrador** en Firebase → Authentication → Users → `canirac@caniraclaguna.mx` | La contraseña estaba escrita en `README.md` y en `config.js`, y ambos son públicos en GitHub (también en el historial). Ya la quité de los archivos, pero quien la haya visto todavía puede entrar al panel hasta que la cambies. |
| 2 | 🔴 **Publicar `firestore.rules`** (Firestore → Reglas → pegar → Publicar) | Sin las reglas nuevas, el bloqueo del catálogo es solo visual: cualquiera podría leer los proveedores directo de la base de datos. |
| 3 | 🟠 Verificar que **Google** y **Correo/contraseña** estén activos en Authentication → Sign-in method | El registro usa Google; el inicio de sesión usa correo + contraseña. |
| 4 | 🟡 Authentication → Templates → idioma **Español** | Para que el correo de “¿Olvidaste tu contraseña?” llegue en español. |
| 5 | 🟡 Avisar a los usuarios actuales | Ya no existe “Entrar como invitado”. Quien entraba antes con Google debe completar su registro (datos + contraseña) y esperar tu autorización. |

---

## 2. Errores encontrados y corregidos

### Seguridad
| Sev. | Problema | Corrección |
|------|----------|------------|
| 🔴 | Contraseña del admin en texto plano en `config.js` y `README.md` (archivos públicos). | Eliminada de ambos; el modo local sin nube ya no acepta una contraseña escrita en el código. |
| 🔴 | El catálogo era de lectura pública (`allow read: if true`): cualquiera, sin sesión, podía descargar todos los proveedores. | Nuevas reglas: solo lo leen personas con acceso **confirmado** o el admin. |
| 🟠 | `adminLogin` aceptaba cualquier correo con “@” y luego mostraba el panel **sin comprobar** que fuera el admin. Con usuarios con contraseña, cualquiera habría visto el panel. | Ahora, si la cuenta no es la del admin, se cierra la sesión y se muestra “Esta cuenta no tiene permisos de administrador”. |
| 🟠 | El catálogo y el panel compartían la misma sesión (mismo dominio): iniciar sesión en uno cerraba o reemplazaba la del otro. | El panel usa su propia instancia de Firebase (`"admin"`), con sesión independiente. |
| 🟡 | Cualquier persona sin sesión podía crear documentos en `equipo` y `pushTokens` (spam). | Ahora requieren sesión y se validan tamaños y campos. |
| 🟡 | Enlaces guardados por el admin (Instagram, web, notificaciones, imágenes) se usaban sin validar: un `javascript:` por error ejecutaría código. Los colores se insertaban sin validar en `style`. | Nuevos `safeUrl()` y `safeColor()` en `ui.js`; aplicados en todo el catálogo. |
| 🟢 | El registro de push tenía un “respaldo” que registraba el service worker de mensajería en la raíz, reemplazando a `sw.js` (se perdía el modo sin conexión). | Respaldo eliminado. |

### Pérdida de datos
| Sev. | Problema | Corrección |
|------|----------|------------|
| 🔴 | **“Cargar lista inicial (62)”** sobrescribía la lista completa con la del Excel: borraba fotos, videos, descripciones y proveedores agregados. | Renombrado a **“Completar lista inicial”**: solo agrega los que falten; nunca borra ni modifica. |
| 🟠 | Si la nube fallaba al abrir el panel, se cargaba la copia local/semilla **sin avisar**; al guardar cualquier cosa, esa copia vieja sobrescribía la nube (mismo riesgo con categorías, slider, notificaciones y apariencia). | Carga “estricta” en el panel: si falla, se avisa y se bloquea el guardado hasta recargar. |

### Funcionamiento
| Sev. | Problema | Corrección |
|------|----------|------------|
| 🟠 | Al cerrar una ficha **tocando fuera** de ella, la página quedaba sin poder desplazarse (`overflow:hidden` no se restauraba). | Corregido; un solo control de bloqueo de desplazamiento para todas las ventanas. |
| 🟠 | Al cambiar la categoría de un proveedor desde el **editor** (o al crear uno nuevo), el proveedor no aparecía en la categoría nueva del catálogo (las listas de IDs de categorías no se actualizaban). | El editor sincroniza las listas de categorías al guardar y al eliminar. |
| 🟡 | El selector de categoría del editor solo mostraba las 11 categorías base, no las creadas por ti. | Ahora muestra todas (base + personalizadas). |
| 🟡 | El panel se cargaba dos veces al iniciar sesión (doble lectura a Firestore). | Protegido para cargar una sola vez. |
| 🟡 | Sin conexión, abrir `admin.html` mostraba el **catálogo** (el service worker respondía con `index.html`). | Ahora responde con la misma página guardada. |
| 🟡 | Estrategia de caché “primero la copia guardada” para el código: tras publicar cambios, el HTML nuevo podía mezclarse con JS/CSS viejos. | Código (HTML/JS/CSS/JSON): primero la red, y la copia solo sin conexión. Imágenes: primero la copia. |
| 🟡 | En tabletas/computadora, los resultados de búsqueda se veían partidos en dos columnas (rejilla dentro de rejilla). | Corregido. |
| 🟡 | Las columnas de categorías tenían anchos desiguales cuando un nombre era largo (“Frutas y Verduras”). | Columnas iguales. |
| 🟡 | En el panel, en teléfono, las 6 pestañas se salían de la pantalla (desplazamiento horizontal de toda la página). | Pestañas deslizables en teléfono; menú lateral en computadora. |
| 🟢 | El botón “Ver proveedor” abría la ficha dos veces. | Corregido. |
| 🟢 | `escapeHtml(0)` devolvía vacío. | Corregido. |
| 🟢 | En el panel, el logo del inicio de sesión no estaba centrado. | Corregido. |

---

## 3. Lo nuevo

### Registro y autorización (lo que pediste)
- **Pantalla de acceso** (reemplaza la portada con “Entrar como invitado”): iniciar sesión, solicitar acceso, recuperar contraseña.
- **Registro con Google** → formulario con nombre completo, fecha de nacimiento, restaurante, puesto, **contraseña y confirmación**
  (mínimo 8 caracteres con letra y número, medidor de seguridad, ver/ocultar, aviso en vivo si no coinciden).
  La contraseña se vincula a la cuenta de Google con Firebase, así el correo queda verificado y el usuario entra después con correo + contraseña.
- **Estados**: “Solicitud en revisión” (se abre sola al confirmar, sin recargar) y “Acceso no autorizado” (con botón a WhatsApp de la Cámara).
- **Panel → Autorizaciones de acceso**: tarjetas con todos los datos (incluye edad calculada), filtros Pendientes / Confirmados / Denegados / Todos,
  búsqueda, **Denegar acceso** / **Confirmar acceso**, eliminar registro, exportar CSV, globo rojo con pendientes, aviso en vivo cuando llega una solicitud
  y contador en el ícono de la app instalada.
- **Revocación inmediata**: si retiras un acceso, el catálogo se cierra al instante en el dispositivo de esa persona.

### Diseño adaptable
- **Teléfono**: formato app (barra inferior, menú lateral, ventanas desde abajo).
- **Computadora (≥ 960 px)**: menú superior con todas las secciones, chip de cuenta, buscador junto al título, rejillas de 3–9 columnas,
  destacados en cuadrícula, pie de página, pantalla de acceso a dos columnas y panel admin con menú lateral.
- **Sin zoom manual**: viewport sin escalado + bloqueo de pellizco/doble toque (iPhone ignora el viewport) y de Ctrl + rueda / Ctrl +/−.
  Se evita también el zoom automático del iPhone al enfocar campos.

### App web (PWA) para catálogo **y** panel
- Manifiesto propio del panel (`admin-manifest.json`) → se instala como app separada (“Admin CANIRAC”).
- Botón **Instalar app** (menú del catálogo y barra del panel); en iPhone muestra las instrucciones.
- Ambas funcionan sin conexión con la última copia; una persona ya autorizada puede abrir el catálogo sin internet.

### Mejoras extra
- **Botón «atrás» de Android/navegador**: cierra la ventana abierta o regresa a la vista anterior, en vez de salir de la app.
- **Compartir proveedor** (menú nativo del teléfono o copiar al portapapeles).
- **Teléfono del proveedor “toca para llamar”.**
- Botón para borrar la búsqueda; saludo con el nombre del usuario; mensaje claro cuando “Favoritos” está vacío.
- WhatsApp del proveedor: si escribes 10 dígitos en el editor, se agrega el 52 automáticamente.
- El catálogo guarda copia local de proveedores para uso sin conexión.

---

## 4. Pendientes que recomiendo revisar juntos

| Sev. | Tema | Detalle |
|------|------|---------|
| 🟠 | Carpeta `functions/` ausente | `firebase.json` y `GUIA-PUSH.md` la usan para enviar los avisos push, pero no está en el repositorio. Si ya publicaste la función desde el zip, sigue funcionando; si no, los avisos push no salen. `firebase deploy` (sin `--only`) fallará. |
| 🟡 | Datos de proveedores en `seed.js` | La lista inicial (62 proveedores con teléfonos) está en un archivo público. Las reglas protegen la base de datos, pero ese archivo se puede abrir directamente. Si es un problema, conviene quitarlo del sitio una vez cargada la nube. |
| 🟡 | Fotos y videos (Storage) | Siguen siendo de lectura pública por URL (las URL son difíciles de adivinar). Restringirlas a usuarios autorizados es posible, pero rompe imágenes ya compartidas en notificaciones. |
| 🟡 | Accesibilidad del zoom | Desactivar el zoom dificulta la lectura a personas con baja visión (pauta WCAG 1.4.4). Lo dejé como pediste; los textos base son ≥ 16 px. El zoom desde el menú del navegador en computadora no se puede bloquear. |
| 🟢 | Íconos de la app | `icon-192.png`, `icon-512.png`, `icon-maskable-512.png` y `apple-touch-icon.png` son la misma imagen. El ícono “maskable” necesita margen de seguridad o Android lo recorta. |
| 🟢 | Archivos sin uso | `opticosdemexico.mp4` (1.2 MB), `whats.png`, `ig.png` no se usan en ninguna página. |
| 🟢 | Datos en un solo documento | Todos los proveedores viven en un documento de Firestore (límite 1 MB). Con fotos en Storage está bien; si se agregan fotos en base64 (sin Storage) el límite se alcanza pronto. |
| 🟢 | Aviso al admin por cada solicitud | Hoy te enteras de solicitudes nuevas al abrir el panel. Con la Cloud Function podrías recibir un push o correo por cada solicitud. |

---

## 4-bis. Cambios del 20/oct (segunda ronda)

| Tema | Detalle |
|------|---------|
| 🟠 “Denegación” al registrarse | Causas: (1) tras registrarse, el catálogo quedaba bloqueado hasta la autorización; (2) la regla exigía `email_verified` en el token, que puede faltar justo después de vincular la contraseña a la cuenta de Google → `permiso denegado`. Ahora la regla acepta correo verificado **o** cuenta vinculada a Google, se renueva el token antes de guardar, y los errores muestran su código entre corchetes para diagnosticar. |
| Prueba de 15 días | Acceso inmediato al registrarse. Fecha de inicio puesta por el **servidor** (`inicioPrueba`), validada en las reglas; el usuario no puede alargarla. Las solicitudes anteriores cuentan desde su `createdAt`. |
| Aviso al admin | Cloud Function `avisoSolicitudAcceso` (carpeta `functions-accesos`, codebase `accesos`) + botón “Activar” en el panel + aviso del sistema si el panel está abierto. |
| Pantalla de carga | Logo de CANIRAC + barra con porcentaje real por etapas (conexión, sesión, acceso, proveedores) en catálogo y panel. |
| Mini tutorial de instalación | iPhone / Android / computadora, con detección automática del dispositivo y botón “Instalar ahora”. |

---

## 4-ter. Rediseño y tercera auditoría (22/oct)

**Nuevo diseño** (según la maqueta, con la paleta oficial CANIRAC: teal 3145 C, naranja 158 C, rojo 180 C, gris Cool Gray 2C):
encabezado oscuro con logo blanco y menú (Inicio · Proveedores · Categorías · Favoritos · Nosotros · Contacto), buscador y botón de cuenta naranja;
portada con foto y título con tipografía serif; categorías con foto montadas sobre la portada; “Aliados de confianza para tu negocio” con tarjetas
horizontales (foto, logo, categoría, descripción, ubicación); nueva sección **Nosotros**; pantalla de acceso, carga y panel con la misma paleta.

**Fotos editables desde el panel:** portada del inicio y sus textos (Apariencia → Portada del inicio), foto de cada categoría (Categorías),
foto de portada y ubicación de cada proveedor (editor del proveedor). Si una foto falta o no carga, se muestra el ícono de la categoría.

| Sev. | Error encontrado | Corrección |
|------|------------------|------------|
| 🟠 | **App instalada: se quedaba azul y luego cambiaba a naranja.** El color de “Apariencia” se aplicaba solo al terminar de descargar la configuración, y además reemplazaba el fondo del encabezado; antes de eso se veía el azul por defecto. | La paleta CANIRAC ya viene en el diseño (sin azul), el color de acento ya no toca el encabezado, y el tema se guarda en el teléfono y se aplica **antes** de pintar la página. |
| 🟠 | **Tardaba en cargar al abrir la app.** Las librerías de Firebase (~500 KB) se descargaban en cada apertura y proveedores/configuración se pedían uno tras otro. | El service worker guarda las librerías y tipografías; proveedores y configuración se piden al mismo tiempo; y si la persona ya tenía acceso, el catálogo se muestra **al instante** con la última copia mientras se verifica la sesión. |
| 🟡 | No había forma de recargar la app instalada sin cerrarla. | Botón **Actualizar** en el menú (☰) y en el panel; **jalar hacia abajo** en la app instalada; y al volver a la app después de 5 min se actualizan los datos solos. |
| 🟡 | La lista de categorías del panel no tenía estilos (`card-grid`/`media-card` no existían). | Estilos agregados. |
| 🟡 | Las categorías y proveedores guardados con el azul anterior seguían saliendo en azul. | Se convierten automáticamente al teal CANIRAC. |
| 🟢 | Tabla de proveedores del panel muy larga en teléfono. | Filas compactas. |

---

## 4-quater. Regreso al diseño anterior + logo + compatibilidad (26/oct)

Se restauró el **diseño anterior** (encabezado de color con el logo al centro, buscador, mosaico de categorías y tarjetas de destacados),
conservando todos los arreglos posteriores: sin destello azul (el naranja CANIRAC es el color por defecto y el tema se aplica antes de pintar),
carga rápida (librerías guardadas en el teléfono, datos en paralelo, arranque instantáneo), botón **Actualizar** y jalar para actualizar.
El diseño tipo maqueta quedó respaldado (no se publica).

| Sev. | Hallazgo | Corrección |
|------|----------|------------|
| 🟠 | **Logos subidos en el panel salían con fondo blanco:** toda imagen se convertía a JPG, que no tiene transparencia. | Los logos (del sitio y de proveedores) se guardan en **PNG**. |
| 🟡 | Logo del inicio chico y dentro de una placa blanca. | Sin fondo, en blanco, 100 px por defecto y **tamaño/estilo ajustables** en *Apariencia → Logo del inicio* (vista previa en vivo). Se aplica desde el primer instante. |
| 🟡 | Con un logo grande, el encabezado fijo ocupaba mucha pantalla en el celular. | El encabezado se **compacta al bajar** y vuelve a su tamaño arriba. En teléfono acostado, el logo se limita. |
| 🟡 | iPhone: el buscador del panel tenía letra de 15 px → el iPhone hacía zoom al tocarlo. | Todos los campos de captura tienen ≥ 16 px (revisado en catálogo y panel). |
| 🟢 | En celulares, `100vh` no considera las barras del navegador (la pantalla “brincaba”). | Se usa `100dvh` (con respaldo). |
| 🟢 | Zona segura del iPhone (notch) en la barra inferior y el contenido. | Márgenes con `safe-area`. |

**Compatibilidad revisada** (sin desplazamiento lateral, sin elementos cortados, sin campos que provoquen zoom):
celular 320 px (iPhone SE), 375 px, tableta 768 px, tableta/laptop 1024 px y computadora 1440 px; catálogo, pantalla de acceso y panel.
Celular y tableta vertical usan el formato móvil; desde 960 px (iPad horizontal y computadoras) el formato de escritorio.
El zoom manual sigue bloqueado (pellizco, doble toque, Ctrl + rueda).

---

## 5. Cómo se probó
- Sintaxis de todos los JS (y del módulo del panel) verificada con Node.
- Navegador, teléfono (375×812) y computadora (1366×860): pantalla de acceso conectada a Firebase real, formulario de registro,
  catálogo completo (modo de prueba local), fichas, botón «atrás», panel admin y Autorizaciones (confirmar/denegar con datos de prueba locales),
  sin desplazamiento horizontal y sin errores en consola.
- **No probado** (requiere cuentas reales): registro real con Google, vinculación de contraseña, inicio con correo + contraseña,
  y las reglas nuevas en Firestore. Recomendado: después de publicar las reglas, registra una cuenta de prueba, confírmala en el panel
  y entra con correo + contraseña desde un teléfono.
