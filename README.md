# Post-contenido — Unidad 10: Pruebas de Software en Aplicaciones Web

## Descripción
Repositorio del laboratorio de la Unidad 10 de Programación Web —
Séptimo Semestre. Contiene un único proyecto Maven Spring Boot
(catalogo-productos/) con una suite de pruebas unitarias, de
integración y de capa web sobre un catálogo de productos, más una
colección Postman versionada y un umbral de cobertura verificado
con JaCoCo.

## Parte 1 — Pruebas Unitarias con Mockito y de Integración con @DataJpaTest
ProductoServiceTest usa Mockito (@Mock, @InjectMocks) para probar
la lógica de negocio en aislamiento, cubriendo el caso feliz y dos
casos de error (precio inválido, producto no encontrado).
ProductoRepositoryTest usa @DataJpaTest con H2 en memoria para
probar las consultas del repositorio.

## Parte 2 — Pruebas Web con @WebMvcTest, Postman/Newman y JaCoCo
ProductoControllerTest usa @WebMvcTest con MockMvc para probar la
capa web sin levantar el contexto completo. La colección Postman
"API Catálogo de Productos" (postman/ColeccionCatalogo.json)
ejecuta 4 requests con test scripts contra la aplicación en
ejecución real, y puede correrse también con
`newman run postman/ColeccionCatalogo.json --environment postman/env-local.json`.
JaCoCo verifica una cobertura mínima del 70% de líneas sobre todo
el módulo con `mvn verify`.

## Propósito de cada clase de prueba
- **ProductoServiceTest** (`@ExtendWith(MockitoExtension.class)`, `@Mock`, `@InjectMocks`): prueba solo la lógica de negocio; el repositorio se simula para aislar el servicio de la BD y que la prueba sea rápida y determinista.
- **ProductoRepositoryTest** (`@DataJpaTest` + `TestEntityManager`, H2): las consultas solo se validan contra una BD real, por eso aquí no hay mocks; carga solo la capa de persistencia y revierte los datos entre tests.
- **ProductoControllerTest** (`@WebMvcTest` + `MockMvc` + `@MockBean`): valida rutas, códigos HTTP (200, 404, 201, 400) y JSON sin levantar el contexto completo ni un puerto real. No se usa `@SpringBootTest` porque repetiría lo ya probado en servicio y repositorio y sería más lento.

## Orden en que se agregaron las pruebas
1. Unitarias (servicio): base de la pirámide, las más rápidas.
2. Integración (repositorio con H2).
3. Capa web (`@WebMvcTest`).
4. Colección Postman (extremo a extremo por HTTP real).
5. JaCoCo al cierre de la Parte 2, porque solo entonces la suite cubre las tres capas y el 70 % mide el estado real del proyecto.

## Decisiones de diseño
1. H2 se usa también para ejecución local (no solo pruebas): el
   objetivo de la unidad es la escritura de pruebas, no el modelado
   de persistencia contra un motor externo como en la Unidad 8.
2. buscarPorId lanza EntityNotFoundException en vez de retornar
   null, para que el contrato del método sea explícito y verificable
   con assertThrows.
3. ProductoRepositoryTest usa @DataJpaTest en vez de @SpringBootTest
   para aislar la prueba a la capa de persistencia y mantener la
   suite rápida.
4. ProductoControllerTest usa @WebMvcTest en vez de @SpringBootTest
   con TestRestTemplate, para no duplicar cobertura con las pruebas
   de servicio y repositorio ya existentes.
5. La colección Postman se versiona como archivo .json, no como
   test JUnit, porque prueba el contrato HTTP completo como caja
   negra contra un proceso real, un nivel distinto al de MockMvc.
6. JaCoCo se configura al cierre de la Parte 2, no de la Parte 1,
   para medir la cobertura agregada de las tres capas y evitar que
   el umbral del 70% se cumpla artificialmente sobre una suite
   incompleta.
7. Este laboratorio NO incluye Selenium WebDriver: el dominio es una
   API REST pura sin vista Thymeleaf, y fabricar una vista solo para
   justificar Selenium contradiría la pirámide de pruebas. La
   colección Postman cumple el rol de prueba de extremo a extremo
   sobre la superficie real de la aplicación.
8. JaCoCo excluye solo la clase `*Application`: su único código es el `main`, que solo se ejercita levantando todo el contexto. `model` se mide, para que el umbral del 70 % sea real.

## Cómo compilar y ejecutar
1. Clonar el repositorio: `git clone https://github.com/JorgeRicoP/Rico-post1-u10-proweb`
2. Ejecutar `./mvnw spring-boot:run` dentro de catalogo-productos/
3. Ejecutar la suite completa con verificación de cobertura:
   `./mvnw clean verify` (genera el reporte JaCoCo en
   target/site/jacoco/index.html y falla si la cobertura < 70%)
4. Con la aplicación en ejecución, correr la colección Postman:
   `newman run postman/ColeccionCatalogo.json --environment postman/env-local.json`

## Capturas de pantalla
![Reporte de cobertura JaCoCo >= 70%](capturas/jacoco-reporte.png)
![Tests unitarios, de integración y web en verde](capturas/tests-verde.png)
![Postman Runner con 0 failures](capturas/postman-runner.png)
![mvn clean verify final](capturas/verify-verde.png)
![Newman](capturas/newman.png)