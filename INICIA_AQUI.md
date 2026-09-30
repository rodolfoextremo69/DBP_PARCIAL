# INICIA AQUÍ — Cómo estudiar CineBox para DBP

Este proyecto es un **ejemplo educativo completo** para leer, modificar y ejecutar en IntelliJ. Sigue los temas que aparecen en el documento `material_del_curso/snippets_referencia.md`: validación, ModelMapper, paginación y Security con JWT. El archivo original del curso es un conjunto de fragmentos; este ejemplo añade el contexto y las implementaciones faltantes.

## 1. Encender el proyecto

1. Descomprime el ZIP; abre en IntelliJ la carpeta `DBP_CineBox_Backend`, la que contiene `pom.xml`.
2. Selecciona JDK 17 o 21 y permite que Maven descargue las dependencias.
3. En Windows, abre Docker Desktop. En la terminal del proyecto: `docker compose up -d`.
4. Comprueba PostgreSQL: `docker exec -it cinebox-postgres psql -U postgres -d cinebox_db -c "SELECT 1;"`.
5. Ejecuta `src/main/java/com/ejemplo/cinebox/CineboxApplication.java`.
6. Importa `postman/CineBox.postman_collection.json` en Postman. Ejecuta «Login administrador» y luego «Listar géneros» y «Listar películas».

**Valores locales de práctica:** PostgreSQL `localhost:5433`, BD `cinebox_db`, usuario `postgres`, clave `postgres123`. Usuario administrador `admin@cinebox.local`, clave `Admin12345!`. No uses estas credenciales en proyectos publicados. La conexión local está en `src/main/resources/application-local.properties`.

## 2. Qué carpeta abrir y qué pregunta resuelve

| Orden de estudio | Carpeta/archivo | Pregunta que responde |
|---|---|---|
| 1 | `entity/Pelicula.java`, `Genero.java` | ¿Qué tablas/campos/relaciones se guardarán? |
| 2 | `repository/PeliculaRepository.java`, `GeneroRepository.java` | ¿Qué consultas necesito de la BD? |
| 3 | `dto/PeliculaRequest.java`, `PeliculaResponse.java` | ¿Qué JSON llega y qué JSON sale? |
| 4 | `config/ModelMapperConfig.java` | ¿Cómo convierto Request, Entity y Response? |
| 5 | `service/PeliculaService.java`, `GeneroService.java` | ¿Qué reglas verifico antes de guardar o eliminar? |
| 6 | `controller/PeliculaController.java`, `GeneroController.java` | ¿Qué rutas HTTP expondré? ¿Dónde van `@Valid`, `@PathVariable` y `@RequestParam`? |
| 7 | `exception/` | ¿Cómo respondo con HTTP 400, 404 o 409? |
| 8 | `entity/Account.java`, `service/AccountService.java`, `AuthService.java` | ¿Dónde viven los usuarios y cómo verifico contraseñas? |
| 9 | `security/JwtService.java`, `JwtAuthorizationFilter.java`, `config/SecurityConfig.java` | ¿Cómo se firma, lee y autoriza un JWT? |
| 10 | `controller/AuthController.java` | ¿Dónde está la entrada de registro y login? |

Los documentos `GUIA_ESTUDIO_ARCHIVO_POR_ARCHIVO.md` y `SIMULACRO_Y_COMANDOS.md` amplían este mapa.

## 3. Sigue una petición real a través de las capas

Envía `POST /api/peliculas` con JWT ADMIN y este JSON (consulta primero `GET /api/generos` para conocer el ID):

```json
{
  "titulo":"Matrix",
  "director":"Lana y Lilly Wachowski",
  "fechaEstreno":"1999-03-31",
  "precio":29.90,
  "stock":15,
  "codigoCatalogo":"MATRIX-001",
  "sinopsis":"Película de ejemplo",
  "generoId":1
}
```

Recorrido en el código:

1. `JwtAuthorizationFilter`: extrae `Authorization: Bearer <token>` y autentica al usuario de BD.
2. `SecurityConfig`: exige rol ADMIN para POST de películas.
3. `PeliculaController.crear`: transforma JSON en `PeliculaRequest`, aplica `@Valid` y llama al servicio.
4. `PeliculaService.crear`: verifica duplicado, busca el género real mediante `GeneroRepository` y transforma el DTO a `Pelicula` usando ModelMapper.
5. `PeliculaRepository.save`: JPA/Hibernate envía los datos a PostgreSQL.
6. `PeliculaService.toResponse`: devuelve `PeliculaResponse` sin exponer entidades JPA al cliente.
7. `PeliculaController`: responde con HTTP 201.

**Para entender las relaciones:** el cliente proporciona `generoId`, mientras que en la Entity está el campo `Genero genero`. La FK `genero_id` se define con `@ManyToOne` y `@JoinColumn`.

## 4. Qué modificar si en el examen te piden una bodega en vez de películas

- Renombra `Pelicula` a `Producto`; atributos, por ejemplo: `nombre`, `sku`, `precio`, `stock` y `categoria`.
- Renombra `Genero` a `Categoria`; conserva la relación muchos productos → una categoría.
- Repite el patrón Entity → Repository → Request/Response DTO → Service → Controller.
- Mantén los archivos de autenticación (`Account`, `Role`, `AuthService`, `JwtService`) y adapta únicamente las rutas en `SecurityConfig`.
- Cambia la prueba de Postman y vuelve a provocar 400, 401, 403, 404 y 409 para entender por qué surgen.

## 5. Comandos indispensables

```powershell
docker compose up -d
docker compose ps
docker logs cinebox-postgres
docker exec -it cinebox-postgres psql -U postgres -d cinebox_db
```

Para detener sin borrar los datos: `docker compose down`. **No uses `down -v` si quieres conservar la base de datos**.

## 6. Verificar la ejecución

Dentro de IntelliJ: ejecuta `CineboxApplication`, verifica que arranque y prueba el login en Postman. Para ejecutar las pruebas automatizadas: panel Maven → Lifecycle → `test`, o `mvn test` si instalaste Maven. Las pruebas usan H2 y no requieren Docker.

Nota: esta carpeta contiene los tests y el código fuente, pero la presencia de tests no significa que se hayan ejecutado automáticamente en el equipo del usuario.
