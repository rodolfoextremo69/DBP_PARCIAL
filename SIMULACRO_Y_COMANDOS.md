# Simulacro práctico y comandos del examen

## Comandos que debes poder escribir sin consultar

Abre Docker Desktop y, ubicado en la carpeta del `pom.xml`:

```powershell
docker --version
docker compose up -d
docker compose ps
docker logs cinebox-postgres
docker exec -it cinebox-postgres psql -U postgres -d cinebox_db
```

Dentro de psql: `\dt`, `SELECT * FROM generos;`, `\q`.

En IntelliJ: abrir carpeta -> cargar Maven -> elegir Project SDK JDK 17/21 ->
Run `CineboxApplication` -> ver `Started CineboxApplication`.

### Rúbrica de simulacro autogestionado

- [ ] Sé decir qué hace cada dependencia del `pom.xml`.
- [ ] Sin mirar, creo una Entity con `@Id`, `@GeneratedValue`, `@Column`.
- [ ] Creo `Repository extends JpaRepository<MiEntidad, Long>`.
- [ ] Creo DTO Request y Response y activo `@Valid`.
- [ ] Creo `@Service @RequiredArgsConstructor` y hago CRUD.
- [ ] Creo Controller con GET/POST/PUT/DELETE y `ResponseEntity`.
- [ ] Relaciono 2 entidades con `@ManyToOne` + `@JoinColumn`.
- [ ] Agrego paginación con `Pageable` / `Page<T>` / `Page.map`.
- [ ] Sé manejar un 404 y un 409.
- [ ] Registro USER, hago login y recibo JWT.
- [ ] Comprendo Account, UserDetailsService, JwtService, JwtFilter y SecurityFilterChain.
- [ ] Demuestro que USER obtiene 403 al intentar escribir.

## Errores frecuentes

- **Port 5433 is allocated:** edita **compose.yaml** y el **application-local.properties** juntos.
- **Connection refused:** motor Docker detenido / contenedor no listo (`docker compose ps`).
- **Login devuelve 401:** confirma datos demo, revisar si la BD contiene el admin y login JSON.
- **JWT inválido:** usar el token completo sin comillas, encabezado `Authorization: Bearer TOKEN`.
- **Bean Validation no corre:** verifica dependencia starter-validation + `@Valid` en Controller.
- **ModelMapper falla:** asegúrate de crear `@Bean ModelMapper` dentro de una clase `@Configuration`.
- **Lombok subrayado rojo:** vuelve a cargar Maven y habilita procesamiento de anotaciones.
- **Confundes 400/401/403/404/409:** provoca cada caso intencionalmente en Postman.
- **Hibernate crea tablas pero no muestra datos:** revisa el esquema/puerto/base correctos.

## Ejercicio sin copiar: convierte CineBox en una biblioteca

Crea una entidad `Libro(id, titulo, autor, isbn, precio, stock, categoria)`, DTO con
`@NotBlank`, `@Positive` y `@Size`. Define el repositorio y un servicio con CRUD,
búsqueda por título y paginación. Mantén Security: USER consulta y ADMIN modifica.
Compara tu resultado con CineBox archivo por archivo.
