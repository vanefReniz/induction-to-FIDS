# Induction-to-FIDS

1. Bajar sin borrar volúmenes (y subir de nuevo)
Bajar (contenedores y redes se van; volúmenes se quedan):

´´´bash
docker compose -f compose.yaml -f compose.lab.yaml --profile single down
´´´
No uses -v — eso sí borra volúmenes (postgres_data, logos, etc.).

Subir de nuevo (datos siguen ahí):

´´´bash
docker compose -f compose.yaml -f compose.lab.yaml --profile single up -d --wait
´´´
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
