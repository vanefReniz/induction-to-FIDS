# Induction-to-FIDS

Después de configurar el archivo `.env`, logré levantar los contenedores de Docker siguiendo estos pasos (aunque todavía no visualizo información cargada en la interfaz):

## Subir Volumenes 

### 1. Levantar los servicios con Docker Compose

Ubícate en la carpeta del proyecto y ejecuta el siguiente comando para construir y levantar la infraestructura:

```bash
cd ~/ruta/a/baq-new-fids2
docker compose -f compose.yaml -f compose.lab.yaml --profile single up -d --build --wait
```

**Desglose del comando:**

| Parámetro | Función |
| :--- | :--- |
| `docker compose` | Orquesta la aplicación multicontenedor a partir de los archivos Compose. |
| `-f compose.yaml` | Define el stack base (Postgres, Caddy y la aplicación). |
| `-f compose.lab.yaml` | Aplica la configuración de laboratorio (HTTP en `localhost:8080`, cookies sin atributo *Secure* y `APP_ORIGIN` local). |
| `--profile single` | Inicia el servicio de la aplicación (sin esto, solo arrancan Postgres y Caddy). |
| `up` | Crea las redes, volúmenes e inicia los contenedores. |
| `-d` | Ejecuta los contenedores en segundo plano (*detached*) para mantener libre la terminal. |
| `--build` | Reconstruye la imagen de la aplicación desde el `Dockerfile` antes de arrancar. |
| `--wait` | Espera hasta que las comprobaciones de estado (*healthchecks*) pasen o fallen antes de devolver el control. |

**Estado esperado al finalizar:**
* **Postgres** activo con su volumen de datos.
* **Aplicación** (NestJS + React) corriendo en la red interna.
* **Caddy** haciendo de proxy inverso hacia `http://localhost:8080`.

---

### 2. Inicializar datos y crear usuario administrador

Una vez que los contenedores estén corriendo correctamente, ejecuta el script de inicialización para crear el usuario administrador (pedirá un correo y una contraseña de mínimo 14 caracteres):

```bash
./scripts/seed.sh lab
```

**Credenciales creadas para entorno local:**
* **URL de acceso:** `http://localhost:8080`
* **Usuario / Email:** `vanereniz11@gmail.com`
* **Contraseña:** `password1234567`





-------
## Bajar Volúmenes

1. Bajar sin borrar volúmenes (y subir de nuevo)
Bajar (contenedores y redes se van; volúmenes se quedan):

```bash
docker compose -f compose.yaml -f compose.lab.yaml --profile single down
```
No uses -v — eso sí borra volúmenes (postgres_data, logos, etc.).

Subir de nuevo (datos siguen ahí):

```bash
docker compose -f compose.yaml -f compose.lab.yaml --profile single up -d --wait
```
--build solo si cambiaste código. No hace falta volver a hacer seed si el admin/datos ya están en el volumen.

2. ¿Cualquier equipo en la red ve el 8080?
No, no en el lab. compose.lab.yaml publica Caddy solo en:

127.0.0.1:8080

Eso es solo tu Mac. Otro celular/PC en el Wi‑Fi no entra a tu localhost. localhost siempre es “esta máquina”.

3. Despliegue real ≠ lab en localhost
La app no está pensada para vivir solo en tu laptop. El lab es para desarrollo. En aeropuerto/servidor:

Corre en un servidor con Docker (compose.yaml, HTTPS, dominio real, APP_ORIGIN=https://…).
Persistencia: volúmenes Docker en el disco del servidor (postgres_data, logos, content) — no dependen de “localhost”; localhost solo es la URL en tu máquina de lab.
Restricción de red: firewall / red operativa (solo quien deba llegar al servidor), no magia de localhost. Cookies Secure + origen exacto en producción.
Resumen: lab = solo tu Mac + volúmenes locales; producción = servidor en la red del aeropuerto + volúmenes en ese servidor + firewall.
