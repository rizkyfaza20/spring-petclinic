# Spring PetClinic with CI/CD to GCP Instances

## Run Petclinic Locally

Spring Petclinic is a [Spring Boot](https://spring.io/guides/gs/spring-boot) application built using [Maven](https://spring.io/guides/gs/maven/) or [Gradle](https://spring.io/guides/gs/gradle/). You can build a jar file and run it from the command line (it should work just as well with Java 17 or newer).

## Building a Container

There are some Dockerfile contained in this repository. You can try building with either Spring Boot Plugin or Docker.

```bash
./mvnw spring-boot:build-image
```


```bash
docker build -t spring-petclinic .
```


## Database Configuration

In its default configuration, Petclinic uses an in-memory database (H2) which gets populated at startup with data. The h2 console is exposed at `http://localhost:8080/h2-console`, and it is possible to inspect the content of the database using the `jdbc:h2:mem:<uuid>` URL. The UUID is printed at startup to the console.

A similar setup is provided for MySQL and PostgreSQL if a persistent database configuration is needed. Note that whenever the database type changes, the app needs to run with a different profile:
- `spring.profiles.active=mysql` for MySQL 
- `spring.profiles.active=postgres` for PostgreSQL

See the [Spring Boot documentation](https://docs.spring.io/spring-boot/how-to/properties-and-configuration.html#howto.properties-and-configuration.set-active-spring-profiles) for more detail on how to set the active profile.

### Starting a Database

You can start MySQL or PostgreSQL locally with whatever installer works for your OS or use docker:


docker run -e MYSQL_USER=petclinic -e MYSQL_PASSWORD=petclinic -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=petclinic -p 3306:3306 mysql:9.0


or


docker run -e POSTGRES_USER=petclinic -e POSTGRES_PASSWORD=petclinic -e POSTGRES_DB=petclinic -p 5432:5432 postgres:17.0


Further documentation is provided for:
- [MySQL](https://github.com/spring-projects/spring-petclinic/blob/main/src/main/resources/db/mysql/petclinic_db_setup_mysql.txt)
- [PostgreSQL](https://github.com/spring-projects/spring-petclinic/blob/main/src/main/resources/db/postgres/petclinic_db_setup_postgres.txt)

### Using Docker Compose

Instead of vanilla `docker` you can also use the provided `docker-compose.yml` file to start the database containers. Each one has a profile just like the Spring profile:

```bash
docker-compose --profile mysql up
```

or

```bash
docker-compose --profile postgres up
```

## Test Applications

At development time we recommend you use the test applications set up as `main()` methods in:
- `PetClinicIntegrationTests` (using the default H2 database and Spring Boot Devtools)
- `MySqlTestApplication` 
- `PostgresIntegrationTests`

These are set up so that you can run the apps in your IDE to get fast feedback and also run the same classes as integration tests against the respective database. The MySQL integration tests use Testcontainers to start the database in a Docker container, and the Postgres tests use Docker Compose to do the same thing.

## Compiling the CSS

There is a `petclinic.css` in `src/main/resources/static/resources/css`. It was generated from the `petclinic.scss` source, combined with the [Bootstrap](https://getbootstrap.com/) library. If you make changes to the `scss`, or upgrade Bootstrap, you will need to re-compile the CSS resources using the Maven profile "css":

```bash
./mvnw package -P css
```

Note: There is no build profile for Gradle to compile the CSS.

## Working with Petclinic in your CI/CD Process

### Prerequisites

The following items should be in CI/CD:

- GCloud CLI Extension; authorization into GCP resources
- Buildx Docker
- (Optional) Semantic Versioning
- Docker Hub Account

### Setup Steps

> **Warning**: Make sure to run the Infra Petclinic for serving these apps.
<br>
To deploy the Infrastructure please refer to [Infrastructure Repository](https://github.com/rizkyfaza20/infra-petclinic)

1. Clone the Springboot Petclinic repository to integrate with Github Actions:

```bash
git clone https://github.com/rizkyfaza20/spring-petclinic.git
```

2. Configure the application for Docker deployment. You can modify it to use other databases such as Cloud SQL (optional).

3. Set up the required environment variables:
- `DATABASE_URL`
- Database credentials:
  - `POSTGRES_USER`
  - `POSTGRES_DB`
  - `POSTGRES_PASSWORD`
  
Note: If using Cloud SQL, only `DATABASE_URL` is required.

4. Add these variables to Github Secrets via Repository Secrets.

5. After configuring secrets, make changes via PR or push directly to the `main` branch.

6. The CI process will run according to the configuration.

### FAQ

#### GCloud Setup
Q: How do I setup the GCloud Section?
<br>
A: Create a service account in the Google Cloud IAM Section. Generate a JSON Key for credentials to manage GCloud connections. Service Account creation is automated in Infra Petclinic - you only need to create the keys.

#### SSH Configuration
Q: What should I do with SCP and SSH connection?
<br>
A: Create an SSH Key and set:
- `SSH_KEY` variable with the `id_rsa` content
- `SSH_USERNAME` variable
If `SSH_KEY` doesn't work, try using `SSH_PASSWORD` instead.

### Observability
Q: What monitoring I should have to monitor the resources?
<br>
A: In this repository, I use the basic monitoring observability by GCP. If there's any we can use Prometheus and Grafana as an alternative.


