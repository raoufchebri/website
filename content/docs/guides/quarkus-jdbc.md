---
title: Connect Quarkus (JDBC) to Neon
subtitle: Learn how to connect to Neon from Quarkus using JDBC
enableTableOfContents: true
updatedOn: '2024-02-08T15:20:54.288Z'
---

[Quarkus](https://quarkus.io/) is a Java framework optimized for cloud environments. This guide shows how to connect to Neon from a Quarkus project using the PostgreSQL JDBC driver.

To connect to Neon from a Quarkus application using the Postgres JDBC Driver:

1. [Create a Neon Project](#create-a-neon-project)
2. [Create a Quarkus project and add dependencies](#create-a-quarkus-project)
3. [Configure a PostgreSQL data source](#configure-a-postgresql-data-source)
4. [Use the PostgreSQL JDBC Driver](#use-the-postgresql-jdbc-driver)
5. [Run the application](#run-the-application)

## Create a Neon project

If you do not have one already, create a Neon project:

1. Navigate to the [Projects](https://console.neon.tech/app/projects) page in the Neon Console.
2. Click **New Project**.
3. Specify your project settings and click **Create Project**.
4. Once the project is created, save the connection details for later use.

## Create a Quarkus project

1. Ensure you have Java 11+ and Maven 3.8.1+ installed on your system.
2. Install the [Quarkus CLI](https://quarkus.io/guides/cli-tooling) if you haven't already.
3. Create a Quarkus project using the Quarkus CLI:

```shell
quarkus create app neon-with-quarkus-jdbc \
--name neon-with-quarkus-jdbc \
--package-name com.neon.tech \
--extensions jdbc-postgresql,quarkus-agroal,resteasy-reactive
```

This command creates a Quarkus project in a folder named `neon-with-quarkus-jdbc` with the following extensions:
- `jdbc-postgresql`: PostgreSQL JDBC driver
- `quarkus-agroal`: Agroal datasource implementation
- `resteasy-reactive`: RESTEasy Reactive for building REST APIs

4. Navigate to the project directory:

```shell
cd neon-with-quarkus-jdbc
```

5. Verify the dependencies in the `pom.xml` file. Ensure the following dependencies are present:

```xml
<dependencies>
    <dependency>
        <groupId>io.quarkus</groupId>
        <artifactId>quarkus-jdbc-postgresql</artifactId>
    </dependency>
    <dependency>
        <groupId>io.quarkus</groupId>
        <artifactId>quarkus-agroal</artifactId>
    </dependency>
    <dependency>
        <groupId>io.quarkus</groupId>
        <artifactId>quarkus-resteasy-reactive</artifactId>
    </dependency>
</dependencies>
```

## Configure a PostgreSQL data source

1. Create a `.env` file in the root of your Quarkus project directory.
2. Add the following environment variables, replacing the placeholders with your Neon connection details:

```shell
QUARKUS_DATASOURCE_DB_KIND=postgresql
QUARKUS_DATASOURCE_USERNAME=your_neon_username
QUARKUS_DATASOURCE_PASSWORD=your_neon_password
QUARKUS_DATASOURCE_JDBC_URL=jdbc:postgresql://your_neon_hostname/your_neon_database?sslmode=require
```

<Admonition type="note">
You can find the connection string for your database in the **Connection Details** widget on the Neon **Dashboard**. For more information, see [Connect from any application](/docs/connect/connect-from-any-app).
</Admonition>

3. Update the `src/main/resources/application.properties` file to use these environment variables:

```properties
quarkus.datasource.db-kind=${QUARKUS_DATASOURCE_DB_KIND}
quarkus.datasource.username=${QUARKUS_DATASOURCE_USERNAME}
quarkus.datasource.password=${QUARKUS_DATASOURCE_PASSWORD}
quarkus.datasource.jdbc.url=${QUARKUS_DATASOURCE_JDBC_URL}
```

<Admonition type="important">
Never commit the `.env` file to version control. Add it to your `.gitignore` file to prevent accidentally exposing sensitive information.
</Admonition>

## Use the PostgreSQL JDBC Driver

Create a `PostgresResource.java` file in the `src/main/java/com/neon/tech` directory with the following content:

```java
package com.neon.tech;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import javax.sql.DataSource;
import jakarta.inject.Inject;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.Produces;
import jakarta.ws.rs.core.MediaType;

@Path("/postgres")
public class PostgresResource {
    @Inject
    DataSource dataSource;

    @GET
    @Path("/version")
    @Produces(MediaType.TEXT_PLAIN)
    public String getVersion() {
        String query = "SELECT version()";
        try (Connection connection = dataSource.getConnection();
             PreparedStatement statement = connection.prepareStatement(query);
             ResultSet resultSet = statement.executeQuery()) {

            if (resultSet.next()) {
                return resultSet.getString(1);
            }
        } catch (SQLException e) {
            return "Error: " + e.getMessage();
        }
        return "No version information available";
    }
}
```

This code defines an HTTP endpoint that queries the database version and returns it as a response to incoming requests. It uses a `PreparedStatement` for improved security and performance.

## Run the application

1. Start the application in development mode using the Quarkus CLI:

```shell
quarkus dev
```

2. Visit [localhost:8080/postgres/version](http://localhost:8080/postgres/version) in your web browser. Your Neon database's Postgres version will be returned. For example:

```
PostgreSQL 15.4 on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
```

## Security Best Practices

1. Always use environment variables or a secure configuration management system for storing sensitive information like database credentials.
2. Use prepared statements (as shown in the example) to prevent SQL injection attacks.
3. Implement proper error handling and avoid exposing detailed error messages to clients in production.
4. Keep your Quarkus and all dependencies up to date to benefit from the latest security patches.
5. Consider using Quarkus' built-in security features for authentication and authorization in production applications.

<NeedHelp/>
