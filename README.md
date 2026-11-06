# Tourist Record API with Token Validation

A Spring Boot REST API for securely retrieving a tourist record by ID. The custom `Token` header is checked in a Spring MVC interceptor **before the controller/service/repository runs**.

## Requirements
- Java 17 or later
- Eclipse IDE (Enterprise Java and Web Developers package recommended)
- Maven (Eclipse can use its embedded Maven)

## Import into Eclipse
1. Extract `TouristRecordTokenValidation.zip`.
2. Open Eclipse.
3. Select **File -> Import -> Maven -> Existing Maven Projects**.
4. Browse to the extracted `TouristRecordTokenValidation` folder.
5. Select the `pom.xml`, then click **Finish**.
6. Wait for Maven dependencies to download.
7. Right-click `TouristRecordApplication.java` -> **Run As -> Java Application** (or run the project as a Spring Boot App if Spring Tools is installed).

## Demo configuration
The demo token is configured in `src/main/resources/application.properties`:

`app.security.token=TOURIST-ACCESS-2026`

For a real deployment, replace this value and store secrets outside source control (for example, environment variables or a secrets manager).

## API endpoint

`GET http://localhost:8080/api/tourists/{id}`

Required request header:

`Token: TOURIST-ACCESS-2026`

### Example using curl

```bash
curl -i -H "Token: TOURIST-ACCESS-2026" http://localhost:8080/api/tourists/1
```

### Example success response (200)

```json
{
  "id": 1,
  "name": "Ananya Rao",
  "age": 26,
  "email": "ananya@example.com",
  "phone": "+91-9876543210",
  "nationality": "Indian",
  "destination": "Taj Mahal"
}
```

## Response cases

| Situation | HTTP status | Meaning |
|---|---:|---|
| Valid token and existing ID | 200 | Tourist record returned |
| Missing or invalid token | 401 | Request rejected before database access |
| Valid token but unknown ID | 404 | Tourist record does not exist |
| Database or unexpected server error | 500 | Generic error response; details are logged server-side |

## Swagger UI
Start the application and open:

`http://localhost:8080/swagger-ui.html`

OpenAPI JSON:

`http://localhost:8080/api-docs`

The API operation documents the required `Token` header. In Swagger UI, enter the token value as `TOURIST-ACCESS-2026`.

## H2 database console
`http://localhost:8080/h2-console`

Use:
- JDBC URL: `jdbc:h2:mem:touristdb`
- User Name: `sa`
- Password: leave blank

The database is in-memory. The three sample tourist records are loaded each time the application starts.

## Project structure

```text
src/main/java/com/example/tourist/
  TouristRecordApplication.java
  config/
    WebConfig.java
  controller/
    TouristController.java
  exception/
    ApiError.java
    GlobalExceptionHandler.java
    TouristNotFoundException.java
  model/
    Tourist.java
  repository/
    TouristRepository.java
  security/
    TokenValidationInterceptor.java
  service/
    TouristService.java
src/main/resources/
  application.properties
  data.sql
```

## Design notes
- `TokenValidationInterceptor` validates the custom header before protected requests reach the controller. It uses constant-time byte comparison.
- `WebConfig` applies the interceptor to `/api/tourists/**`.
- `TouristController` exposes the GET endpoint.
- `TouristService` contains retrieval logic.
- `TouristRepository` uses Spring Data JPA.
- `GlobalExceptionHandler` maps missing records to 404 and database/unexpected failures to 500.

This is a learning/demo project. For production, use HTTPS, secret management, token rotation/expiry, rate limiting, and an established authentication mechanism where appropriate.
