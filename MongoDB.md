#### Step 1: Add MongoDB Dependency

In your `pom.xml`, add the MongoDB dependency for Spring Boot. This will enable Spring Boot to work with MongoDB.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

#### Step 2: Configure MongoDB Connection in `application.yml`

You need to provide your MongoDB connection details, and since you are using MongoDB (and not an SQL database), you must also exclude Spring Boot’s default SQL configurations.

Here’s the complete `application.yml`:

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            scope:
              - email
              - profile
            redirect-uri: "http://localhost:8080/api/login/oauth2/code/google"

  data:
    mongodb:
      uri: mongodb+srv://<username>:<password>@<cluster-url>/<yourDatabaseName>?retryWrites=true&w=majority

  autoconfigure:
    exclude:
      - org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
      - org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration

server:
  servlet:
    context-path: /api
  port: 8080

logging:
  level:
    root: INFO
    org.springframework: DEBUG
    org.springframework.security: DEBUG
    org.springframework.web: DEBUG
    org.springframework.oauth2: TRACE
    org.springframework.security.oauth2: TRACE
    com.example.demo: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
```

- **Replace** `<username>`, `<password>`, and `<cluster-url>` with your actual MongoDB credentials and cluster information.
- **`spring.autoconfigure.exclude`**: Ensures that Spring Boot does not attempt to configure an SQL `DataSource` or JPA as you are using MongoDB.

#### Step 3: Create a MongoDB Entity

Define an entity class representing a MongoDB document. For example, if you are storing movies in your database:

```java
package com.example.demo.entity;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "movies")
public class Movie {

    @Id
    private String id;
    private String title;
    private String genre;
    private boolean popular;
    private String posterUrl;

    // Constructors, Getters, and Setters
}
```

#### Step 4: Create a MongoDB Repository

Create a repository interface to interact with MongoDB. This interface extends `MongoRepository` and provides CRUD operations.

```java
package com.example.demo.repository;

import com.example.demo.entity.Movie;
import org.springframework.data.mongodb.repository.MongoRepository;
import java.util.List;

public interface MovieRepository extends MongoRepository<Movie, String> {
    List<Movie> findByGenre(String genre);
    List<Movie> findByPopularTrue();
}
```

#### Step 5: Create a Service Layer

Create a service that contains the business logic to interact with the MongoDB repository.

```java
package com.example.demo.service;

import com.example.demo.entity.Movie;
import com.example.demo.repository.MovieRepository;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class MovieService {

    private final MovieRepository movieRepository;

    public MovieService(MovieRepository movieRepository) {
        this.movieRepository = movieRepository;
    }

    public List<Movie> getPopularMovies() {
        return movieRepository.findByPopularTrue();
    }

    public List<Movie> getMoviesByGenre(String genre) {
        return movieRepository.findByGenre(genre);
    }

    public Movie saveMovie(Movie movie) {
        return movieRepository.save(movie);
    }

    public Movie getMovieById(String id) {
        return movieRepository.findById(id).orElse(null);
    }

    public void deleteMovieById(String id) {
        movieRepository.deleteById(id);
    }
}
```

#### Step 6: Create a REST Controller

Create a controller to expose the API endpoints to interact with the Movie entity.

```java
package com.example.demo.controller;

import com.example.demo.entity.Movie;
import com.example.demo.service.MovieService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/movies")
public class MovieController {

    private final MovieService movieService;

    public MovieController(MovieService movieService) {
        this.movieService = movieService;
    }

    @GetMapping("/popular")
    public ResponseEntity<List<Movie>> getPopularMovies() {
        List<Movie> popularMovies = movieService.getPopularMovies();
        return ResponseEntity.ok(popularMovies);
    }

    @GetMapping("/genre/{genre}")
    public ResponseEntity<List<Movie>> getMoviesByGenre(@PathVariable String genre) {
        List<Movie> movies = movieService.getMoviesByGenre(genre);
        return ResponseEntity.ok(movies);
    }

    @PostMapping("/add")
    public ResponseEntity<Movie> addMovie(@RequestBody Movie movie) {
        Movie savedMovie = movieService.saveMovie(movie);
        return ResponseEntity.ok(savedMovie);
    }

    @GetMapping("/{id}")
    public ResponseEntity<Movie> getMovieById(@PathVariable String id) {
        Movie movie = movieService.getMovieById(id);
        if (movie == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(movie);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteMovie(@PathVariable String id) {
        movieService.deleteMovieById(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Step 7: Run the Application

To run the application:

1. Make sure your MongoDB Atlas cluster is up and running.
2. Use the following command:

   ```bash
   mvn spring-boot:run
   ```

Your Spring Boot application should now be connected to MongoDB, and you can interact with it via the REST API endpoints.
