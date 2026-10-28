# Elegant Drops — vitrina técnica

Tienda online real de decants y perfumes para Chile, en funcionamiento en **[elegantdrops.cl](https://elegantdrops.cl)**.
La diseñé, desarrollé, desplegué y opero yo (Diego Miranda), desde abril de 2026.

> El código fuente está en un repositorio privado porque es un negocio activo.
> Este repo resume la arquitectura y las decisiones técnicas; puedo mostrar el código en una revisión técnica.

## Visión general

Un monolito Spring Boot que sirve dos aplicaciones React compiladas dentro del mismo JAR: la **tienda pública** y el **panel de administración**.

| Capa | Tecnología |
|---|---|
| Backend | Java 21, Spring Boot, Spring MVC, Spring Security |
| Datos | MySQL 8, Spring Data JPA, migraciones Flyway (38), caché Caffeine |
| Frontend | React 19 + Vite (dos SPAs), React Router, Vitest |
| Servicios | Cloudinary (imágenes), Brevo (correo transaccional), Mercado Pago (Checkout Pro) |
| Automatización | n8n: ficha olfativa de cada perfume generada con IA y revisada por una persona antes de publicarse |
| Despliegue | Docker multi-stage, imagen en GHCR, GitHub Actions, VPS Linux con Nginx y Cloudflare |

## Qué incluye

- **Catálogo y compra:** decants por formato (ml) con stock real por formato, packs, descuentos, filtros combinables, carrito compartible, checkout y registro de pedidos con correos al comprador y al administrador.
- **Panel de administración:** inventario con orden manual (arrastrar y soltar), ventas, reseñas con moderación, usuarios del personal y registro de ventas manuales vinculado al stock.
- **SEO y rendimiento:** metadatos y datos estructurados dinámicos por producto, Open Graph, imágenes optimizadas y fuentes locales.
- **Analítica y marketing:** Meta Pixel y API de conversiones, Microsoft Clarity, atribución por UTM guardada en cada pedido, con consentimiento de cookies.

## Seguridad

Diseñada con Spring Security y verificada con pruebas automatizadas:

- Cuentas de cliente y de personal con roles jerárquicos (`SUPER_ADMIN > EMPLEADO > CLIENTE`) y verificación de correo obligatoria.
- **Verificación en dos pasos** por correo para el personal, con dispositivos de confianza revocables.
- Sesiones guardadas en base de datos (sobreviven a un despliegue y se pueden cerrar desde «Dispositivos conectados») y regeneración del identificador al iniciar sesión.
- **Reautenticación** para operaciones críticas (responde 428 y exige la contraseña de nuevo).
- Protección CSRF con token por cookie, bloqueo de intentos de acceso por IP y por cuenta, y límite de peticiones por IP.
- Cabeceras de seguridad: HSTS, CSP con nonce por petición, `X-Frame-Options`, `Referrer-Policy`.
- **Precios y disponibilidad recalculados en el servidor:** el navegador nunca decide el monto de un pedido.
- Webhook de pagos autenticado por firma HMAC, validación de archivos subidos (tipo, firma binaria, tamaño y dimensiones) y errores que no filtran detalles internos.
- Secretos fuera del repositorio: variables de entorno y GitHub Secrets.

## Calidad y pruebas

- Más de **300 pruebas automatizadas** en el backend (JUnit, Spring Boot Test, H2): control de acceso, CSRF, integridad del checkout, MFA, reautenticación, webhook de pagos, caché, repositorios y validadores.
- Una prueba de reflexión falla si se agrega un endpoint de administración sin su regla de autorización.
- Pruebas de frontend con Vitest en los flujos de cuenta y carrito.
- Integración continua en GitHub Actions: se ejecutan las pruebas y se construyen ambos frontends antes de publicar la imagen.

## Despliegue

1. Cada `push` a `master` ejecuta pruebas y construye la imagen Docker (frontends incluidos).
2. La imagen se publica en GHCR con un tag por commit.
3. El job de despliegue actualiza el contenedor en el VPS (MySQL nativo, Nginx como proxy inverso, Cloudflare delante).
4. Retroceder a una versión anterior es cambiar el tag de la imagen.

## Contacto

Diego Miranda · Santiago, Chile · [elegantdrops.cl](https://elegantdrops.cl)
