PixelPro🎮 – Hito 2: PHP/PDO, CRUD y Autenticación

Sitio web de PixelPro (empresa de videojuegos) con conexión a MySQL mediante PDO, un módulo CRUD de juegos para administradores y un sistema de registro / login con control de acceso por roles.

1. Características
Conexión segura con PDO y manejo de excepciones (try-catch).
Consultas preparadas (:param) en todos los INSERT, SELECT, UPDATE y DELETE.
Prevención de XSS: toda variable impresa pasa por htmlspecialchars() (helper e()).
Registro con correo Gmail y contraseña hasheada con password_hash().
Login con sesiones PHP (session_start()), session_regenerate_id() y límite de intentos fallidos.
Roles: Administrador y Jugador.
Administrador predeterminado creado automáticamente.
CRUD de juegos (solo administrador).
Gestión de usuarios (solo administrador): cambiar contraseñas y eliminar cuentas.
Protección CSRF en todos los formularios que modifican datos.
2. Requisitos
XAMPP o Laragon (Apache + MySQL/MariaDB)
PHP 7.4 o superior (recomendado 8.x)
MySQL 5.7+ / MariaDB
3. Estructura del proyecto
pixelpro/
├── config/
│   ├── config.php         # Constantes (BASE_URL, admin, roles)
│   ├── conexion.php       # Conexión PDO con try-catch
│   └── inicializar.php    # Crea roles, catálogos base y el admin
├── includes/
│   ├── funciones.php      # Sesión, e() anti-XSS, CSRF, permisos
│   ├── header.php         # Plantilla: apertura del HTML
│   ├── nav.php            # Menú (cambia según la sesión)
│   └── footer.php         # Plantilla: cierre del HTML
├── auth/
│   ├── login.php          # Inicio de sesión
│   ├── registro.php       # Registro de usuarios
│   └── logout.php         # Cierre de sesión (POST + CSRF)
├── admin/
│   ├── index.php          # READ    – listado de juegos
│   ├── crear.php          # CREATE  – nuevo juego
│   ├── editar.php         # UPDATE  – editar juego
│   ├── eliminar.php       # DELETE  – borrar juego (POST + CSRF)
│   └── usuarios.php       # Gestión de cuentas de usuario
├── css/
│   ├── todo.css           # Estilos originales del sitio
│   └── admin.css          # Alertas, tablas y botones
├── js/
│   └── todo.js            # Scripts originales del sitio
├── sql/
│   └── schema.sql         # Estructura de la base de datos
├── img/                   # (tus imágenes)
└── *.php                  # (tus páginas: blog, noticias, generos, etc.)
4. Instalación
Copiar el proyecto
XAMPP: C:\xampp\htdocs\pixelpro
Laragon: C:\laragon\www\pixelpro
Agregar las páginas y recursos originales en la raíz de pixelpro/: blog.php, noticias.php, generos.php, ranking.php, precios.php, contactos.php, curso.php, etc., y la carpeta img/. Renombrar los archivos con espacios usando guiones bajos (deseas_unirte.php, perfil_jugador.php, ubicacion_sucursal.php).
Iniciar los servicios: Apache y MySQL.
Crear la base de datos
Abrir http://localhost/phpmyadmin
Pestaña Importar → seleccionar sql/schema.sql → Importar.
No importar consultas.sql (contiene hashes de ejemplo que no sirven para iniciar sesión).
Configurar (solo si hace falta):
config/conexion.php: usuario y contraseña de MySQL (por defecto root / vacío).
config/config.php: BASE_URL debe coincidir con el nombre de la carpeta (por defecto /pixelpro).
Abrir http://localhost/pixelpro/auth/login.php.
5. Credenciales del administrador
Campo	Valor
Correo	alexisthomasayala@gmail.com
Contraseña	123456789

El administrador se crea automáticamente la primera vez que se carga cualquier página (función inicializar_datos_base()). La contraseña se guarda hasheada y no se vuelve a sobrescribir, de modo que si se cambia desde el panel, el cambio se conserva.

⚠️ 123456789 es una contraseña débil, aceptable solo para la entrega académica. Cambiarla desde Usuarios antes de publicar el sitio en un servidor real.

6. Permisos por rol
Acción	Visitante	Jugador	Administrador
Ver páginas públicas (blog, noticias…)	✅	✅	✅
Registrarse / iniciar sesión	✅	—	—
Panel CRUD de juegos (admin/)	❌	❌ (403)	✅
Gestionar usuarios / cambiar contraseñas	❌	❌ (403)	✅
7. Medidas de seguridad aplicadas
Amenaza	Medida
Inyección SQL	Consultas preparadas con parámetros nombrados y EMULATE_PREPARES = false
XSS	htmlspecialchars(ENT_QUOTES, 'UTF-8') en toda salida (e())
Contraseñas	password_hash() / password_verify() / password_needs_rehash()
Fijación de sesión	session_regenerate_id(true) al iniciar sesión
Robo de cookie	Cookie de sesión HttpOnly + SameSite=Lax
CSRF	Token por sesión validado con hash_equals() en todos los POST
Borrado por enlace	Eliminar solo por POST + token (nunca por GET)
Fuerza bruta	5 intentos fallidos → bloqueo de 60 segundos
Enumeración de correos	Mensaje de error genérico y verificación de hash en tiempo constante
Fuga de errores	Los detalles técnicos van a error_log, no al navegador
Acceso indebido	requerir_admin() en cada página de admin/
8. Guía de pruebas para la presentación
Login admin con las credenciales de la sección 5.
Create: + Agregar Nuevo Juego → completar y guardar.
Read: verificar el juego en la tabla.
Update: Editar → modificar y actualizar.
Delete: Eliminar → confirmar.
Registro: cerrar sesión → Registrarse con un Gmail de prueba.
Permisos: iniciar sesión con ese usuario e intentar abrir /admin/index.php → debe mostrar 403.
Gestión de usuarios: como admin, menú Usuarios → cambiar la contraseña del usuario de prueba y comprobar que puede entrar con la nueva.
9. Solución de problemas
Síntoma	Causa probable / solución
"No se pudo conectar con la base de datos"	MySQL apagado o schema.sql sin importar
Sin estilos o enlaces con error 404	BASE_URL no coincide con el nombre de la carpeta
Pantalla en blanco	Error de PHP oculto: revisar xampp/apache/logs/error.log
No entra con el admin	Ya existía un usuario con ese correo: borrarlo desde phpMyAdmin (tabla usuarios) y recargar el login
El registro no envía el formulario	No usar id="formRegistro": todo.js intercepta ese id con preventDefault()
"Solicitud inválida"	Token CSRF vencido: recargar la página e intentar de nuevo
10. Base de datos

Modelo normalizado hasta 3FN (ver NORMALIZACION.md). Tablas: roles, generos_usuario, categorias_juego, tipos_publicacion, idiomas, usuarios, juegos, publicaciones, comentarios, solicitudes_union, solicitud_idioma.

El CRUD de esta entrega trabaja sobre juegos (con JOIN a categorias_juego) y usuarios (con JOIN a roles).


<img src='fotostp/tp7.png' alt='panel de administracion' width='500' height='350'>
