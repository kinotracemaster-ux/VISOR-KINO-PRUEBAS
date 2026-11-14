# Conectar el visor de pruebas a Drive y a la hoja

El código es el mismo que producción: no hace falta programar nada, solo variables en Railway
(servicio del visor de **pruebas**) y conectar Google una vez desde el admin. No subas secretos al repo.

## Cómo se conecta
- **Hoja maestra**: se lee como CSV público (`docs.google.com/spreadsheets/d/{GOOGLE_SHEET_ID}/export`). Solo lectura,
  sin OAuth. La hoja debe estar compartida como "cualquiera con el enlace puede ver".
- **Drive**: OAuth (cliente de Google Cloud). El token queda en la tabla `google_tokens` de la MySQL de **pruebas**
  (no se copia de producción). El sync recorre `GOOGLE_DRIVE_FOLDER_ID` y guarda todo en `drive_index`.
- El sync corre solo al arrancar y cada 60 min (`scripts/sync_loop.sh`): carpetas → hoja → portadas.

## Variables en Railway (servicio de pruebas)
| Variable | Valor |
|---|---|
| `APP_URL` | `https://<dominio-de-pruebas>.up.railway.app` |
| `APP_SECRET` | aleatorio nuevo (`openssl rand -hex 32`), distinto de producción |
| `ADMIN_USER` / `ADMIN_PASSWORD` | los del admin de pruebas |
| `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` | los mismos de producción (mismo cliente OAuth) |
| `GOOGLE_REDIRECT_URI` | `https://<dominio-de-pruebas>.up.railway.app/admin/google/callback` |
| `GOOGLE_DRIVE_FOLDER_ID` | carpeta PRODUCTOS de producción, o una carpeta de prueba |
| `GOOGLE_SHEET_ID` | ID de la hoja MAESTRA (o una copia de prueba); `GOOGLE_SHEET_GID` opcional (pestaña) |
| `DRIVE_READONLY` | `1` si usa la carpeta real: bloquea subir/borrar desde Admin → Media |
| `CRON_SECRET`, `WEBHOOK_SHEETS_SECRET` | aleatorios nuevos (opcionales) |
| `JINA_API_KEY` | opcional (búsqueda por foto) |
| MySQL | agregar el plugin MySQL al proyecto de pruebas; `MYSQL_URL` / `MYSQLHOST…` llegan solas |

## Pasos
1. Google Cloud Console → APIs y servicios → Credenciales → el cliente OAuth de producción →
   **URI de redireccionamiento autorizados**: agregar la `GOOGLE_REDIRECT_URI` de pruebas (sin quitar la de producción).
   Si la pantalla de consentimiento está en modo "Prueba", el correo que conecte debe estar en "Usuarios de prueba".
2. Railway → servicio de pruebas → Variables: poner las de la tabla. Se redespliega solo.
3. Entrar a `/admin` de pruebas → Media → **Conectar Google Drive** → aceptar todas las casillas.
4. Admin → Sincronizar (o esperar el sync automático): primero la hoja, luego portadas. El primer sync completo tarda.

## Ojo con usar la carpeta real
- El sync solo lee Drive, salvo que marca como "cualquiera con el enlace" los archivos nuevos (lo mismo que hace producción).
- Subir y borrar en Admin → Media sí tocan Drive: por eso `DRIVE_READONLY=1`.
- La hoja nunca se escribe desde el visor.
