# CineBox — Backend completo de DBP (Spring Boot + JWT)

Proyecto **educativo** de administración de películas para analizar qué se escribe en cada capa.

**Si es la primera vez que lo abres:** comienza por [`INICIA_AQUI.md`](INICIA_AQUI.md). Para estudiar cada clase usa [`GUIA_ESTUDIO_ARCHIVO_POR_ARCHIVO.md`](GUIA_ESTUDIO_ARCHIVO_POR_ARCHIVO.md). Se conserva como referencia el archivo de snippets suministrado por el alumno en `material_del_curso/snippets_referencia.md` (no es código ejecutable).
Inspirado en los temas del archivo `snippets.md` de DBP: Bean Validation, ModelMapper,
paginación con Pageable/Page, Account implementando UserDetails y Security JWT.
Se completan las partes que en los apuntes originales estaban representadas por `...`.

## 1. ¿Qué implementa?

- Dos CRUD completos: **películas** y **géneros** (relación `@ManyToOne` / `@OneToMany`).
- Validaciones `@NotBlank`, `@NotNull`, `@Size`, `@DecimalMin`, `@Min`, `@Pattern`, `@PastOrPresent`, `@Email`.
- Entity -> DTO y DTO -> Entity mediante `ModelMapper`.
- Búsqueda combinada por título/género, paginación (`Pageable`) y ordenación (`sort`).
- Registro de cuentas y login, hash de contraseñas con BCrypt, JWT firmado, filtro Bearer.
- `USER`: puede consultar. `ADMIN`: puede consultar y modificar películas/géneros.
- Manejo de errores HTTP: 400, 401, 403, 404, 409; además de 201 y 204.
- PostgreSQL 17 mediante Docker Compose; perfil `local` con datos de demostración.
- Pruebas con JUnit + MockMvc usando H2 bajo perfil `test`.
- Colección Postman importable en `postman/`.

## 2. Requisitos

- JDK 17 o 21 (IntelliJ configurado con ese SDK).
- IntelliJ IDEA con soporte Maven, conexión a internet para descargar dependencias una vez.
- Docker Desktop iniciado (backend se ejecuta **en IntelliJ**; Docker aloja PostgreSQL).
- Postman recomendado para probar rutas.

> Se usa Spring Boot **3.5.16** para mantener las convenciones de Spring Security 6 del curso.
> Esta rama ya terminó su soporte OSS general; para proyectos nuevos de producción evalúa
> una línea vigente de Spring Boot. ModelMapper 3.2.4, JJWT 0.13.0.

## 3. Iniciar en Windows: cuatro pasos

**PASO A.** Extrae el ZIP. Abre IntelliJ > `Open` > selecciona la carpeta que contiene `pom.xml`.
Espera a que Maven importe y descargue dependencias. Activa el procesamiento de anotaciones
si tu instalación de IntelliJ lo requiere para Lombok.

**PASO B.** Inicia Docker Desktop. Abre la terminal en la raíz del proyecto:

```powershell
docker compose up -d
docker compose ps
```

El contenedor se llama `cinebox-postgres` y ofrece PostgreSQL en **localhost:5433**.
Se usa 5433 para evitar colisión si en Windows ya está instalado PostgreSQL en 5432.
Puedes verificar la conexión:

```powershell
docker exec -it cinebox-postgres psql -U postgres -d cinebox_db -c "SELECT 1;"
```

**PASO C.** Ejecuta `CineboxApplication.java` (botón verde Run) desde IntelliJ.
El perfil **local** es el predeterminado: crea/actualiza el esquema (`ddl-auto=update`)
y carga géneros, una película y un ADMIN de demostración. No debes crear tablas a mano.
Alternativa usando Maven instalado: `mvn spring-boot:run`.

**PASO D.** Importa `postman/CineBox.postman_collection.json` en Postman y ejecuta
primero `01 - Autenticacion > Login administrador`. El script guarda `adminToken`.
También puedes ejecutar después Registro USER y Login USER para guardar `userToken`.

### Usuario demo (¡solo en perfil local!)

| Campo | Valor |
|---|---|
| Email ADMIN | `admin@cinebox.local` |
| Clave ADMIN | `Admin12345!` |
| URL base | `http://localhost:8080` |
| DB | `cinebox_db` |
| Host y puerto DB | `localhost:5433` |
| DB user/clave | `postgres` / `postgres123` |

Estas son **credenciales ficticias de desarrollo**, no deben emplearse en un servidor real.
Puedes cambiar `DEMO_ADMIN_PASSWORD`, `DB_PASSWORD` y `JWT_SECRET` mediante variables de entorno;
si cambias la contraseña del contenedor, ten presente que `POSTGRES_PASSWORD` solo se aplica
al inicializar un volumen nuevo. El JWT local usa una clave **de demostración** incluida en
`application-local.properties`; el perfil `prod` exige el secreto real por entorno.

## 4. Endpoints y permisos

| HTTP | Ruta | Permiso | Resultado |
|---|---|---|---|
| POST | `/api/auth/register` | público | crea cuenta USER (201) |
| POST | `/api/auth/login` | público | devuelve accessToken (200) |
| GET | `/api/generos` | USER/ADMIN | lista géneros |
| GET | `/api/generos/{id}` | USER/ADMIN | obtiene género |
| POST | `/api/generos` | ADMIN | crea género (201) |
| PUT | `/api/generos/{id}` | ADMIN | actualiza género |
| DELETE | `/api/generos/{id}` | ADMIN | borra si no tiene películas (204) |
| GET | `/api/peliculas` | USER/ADMIN | busca y pagina |
| GET | `/api/peliculas/{id}` | USER/ADMIN | obtiene película |
| POST | `/api/peliculas` | ADMIN | crea película (201) |
| PUT | `/api/peliculas/{id}` | ADMIN | actualiza película |
| DELETE | `/api/peliculas/{id}` | ADMIN | elimina película (204) |

La consulta acepta `/api/peliculas?titulo=inter&generoId=1&page=0&size=5&sort=precio,desc`.
**Nota:** `page=0` es la primera página. Sin token: 401. Con USER intentando editar: 403.

## 5. JSON de ejemplo para probar

`POST http://localhost:8080/api/auth/login` (sin token):

```json
{"email":"admin@cinebox.local","password":"Admin12345!"}
```

Copia `accessToken` de la respuesta. En Postman, para consultas protegidas agrega:

```http
Authorization: Bearer PEGAR_AQUI_EL_TOKEN
```

`POST /api/peliculas` como ADMIN (género 1 corresponde a Ciencia ficción en una BD recién creada):

```json
{
  "titulo": "Matrix",
  "director": "Lana y Lilly Wachowski",
  "fechaEstreno": "1999-03-31",
  "precio": 29.90,
  "stock": 15,
  "codigoCatalogo": "MATRIX-001",
  "sinopsis": "Película de ciencia ficción.",
  "generoId": 1
}
```

Para confirmar el ID real, llama antes a `GET /api/generos` (la colección de Postman
puede guardarlo automáticamente). `codigoCatalogo` es único.

## 6. Orden ideal para leer las clases

1. `entity/Genero.java`, `entity/Pelicula.java` (tablas y relaciones).
2. `repository/` (persistencia sin escribir SQL en el Controller).
3. `dto/` (entrada, salida y Bean Validation).
4. `config/ModelMapperConfig.java` (mapeos de campos homónimos).
5. `service/GeneroService.java` y `service/PeliculaService.java` (negocio).
6. `controller/GeneroController.java` y `controller/PeliculaController.java` (REST).
7. `exception/` (404/409/400).
8. `entity/Account.java`, `security/JwtService.java`, `service/AccountService.java`,
   `service/AuthService.java`, `security/JwtAuthorizationFilter.java` y `config/SecurityConfig.java`.
9. `src/test/.../CineboxIntegrationTest.java` y colección Postman.

**Para entender cada archivo y los imports/anotaciones revisa `GUIA_ESTUDIO_ARCHIVO_POR_ARCHIVO.md`.**

## 7. Pruebas

En una computadora con Maven instalado: `mvn test` (usa H2 y no necesita Docker).
Si la IDE tiene Maven integrado: ventana **Maven > Lifecycle > test**.
Si fallan compilación/imports, comprueba el JDK 17/21, la descarga de dependencias y Lombok.

## 8. Problemas comunes

- `Connection refused`: Docker cerrado, contenedor detenido o puerto equivocado. Ejecuta `docker compose ps`.
- `Port 5433 already allocated`: cambia `5433:5432` por `5434:5432` en `compose.yaml`
  **y cambia el puerto en** `application-local.properties`.
- `Lombok: cannot find symbol get...`: habilita annotation processing y recarga Maven.
- `401`: no llegó un JWT válido / venció; vuelve a hacer login.
- `403`: autenticado pero sin rol ADMIN para editar.
- `409`: duplicado de email/código/género, o intento de borrar un género en uso.
- `400`: validación del DTO o JSON inválido; consulta `details` en el cuerpo de error.
- `DELETE 204`: éxito sin cuerpo de respuesta, es correcto.
- No ejecutes `docker compose down -v` si deseas conservar el contenido del volumen.

## 9. Crear tu propio repositorio a partir del ZIP (opcional)

Este ZIP **no está publicado en GitHub**, así que no tiene URL real de `git clone`.
Después de descomprimirlo, crea un repositorio vacío en tu cuenta y ejecuta:

```powershell
git init
git add .
git commit -m "CineBox backend de práctica DBP"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/cinebox-dbp.git
git push -u origin main
```

Sustituye `TU_USUARIO` por tu cuenta; no subas `.env` ni contraseñas reales.
