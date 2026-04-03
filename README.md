# JPA Many-to-Many Relationship with Spring Boot and Hibernate

## Overview

This project provides a clear and concise example of how to implement a many-to-many relationship between two entities using Spring Data JPA and Hibernate. It demonstrates a common use case where multiple `User` entities can be associated with multiple `Role` entities, and vice-versa.

The core objective is to showcase:
- The configuration of `@ManyToMany` annotations on both sides of the relationship.
- The use of `mappedBy` to define the owning and inverse sides of the relationship, avoiding redundant mapping.
- The implementation of a service layer to manage the entities and their associations transactionally.
- The automatic generation of a join table by Hibernate.

## Technologies Used

- **Java 17**
- **Spring Boot**
- **Spring Data JPA**
- **Hibernate**
- **H2 Database** (In-memory)
- **Lombok**
- **Maven**

## Key Components

### 1. Entities

The model consists of two entities, `User` and `Role`, linked by a many-to-many relationship.

- **`User.java`**: Represents a user in the system. It is the *inverse* (non-owning) side of the relationship, indicated by the `mappedBy` attribute.

  ```java
  @Entity
  @Table(name = "USERS")
  public class User {
      @Id
      private String userId;
      @Column(name = "USER_NAME", unique = true , length = 20)
      private String username;
      private String password;
      @ManyToMany(mappedBy = "users", fetch = FetchType.EAGER)
      private List<Role> roles = new ArrayList<>();
  }
  ```

- **`Role.java`**: Represents a role that can be assigned to users. It is the *owning* side of the relationship. Hibernate will create a join table (e.g., `role_users`) to manage the associations.

  ```java
  @Entity
  public class Role {
      @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
      private Long id;
      @Column(length = 20, unique = true)
      private String roleName;
      @ManyToMany(fetch = FetchType.EAGER)
      private List<User> users = new ArrayList<>();
  }
  ```

### 2. Service Layer

The `UserService` and its implementation `UserServiceImpl` provide the business logic to manage users, roles, and their relationships. The `addRoleRoUser` method demonstrates how to associate a role with a user within a transactional context.

```java
@Service
@Transactional
public class UserServiceImpl implements UserService {
    // ... repositories
    
    @Override
    public void addRoleRoUser(String username, String roleName) {
        User user = findUserByUserName(username);
        Role role = findRoleByRoleName(roleName);
        if(user.getRoles() != null){
            user.getRoles().add(role);
            role.getUsers().add(user);
        }
    }
}
```

### 3. Data Initialization

The `JpaEnsetApplication.java` includes a `CommandLineRunner` bean that initializes the database with a sample user on application startup.

```java
@Bean
CommandLineRunner start(UserService userService){
    return args->{
        User u = new User();
        u.setUsername("user1");
        u.setPassword("123456");
        userService.addNewUser(u);
    };
}
```

## Getting Started

### Prerequisites

- JDK 17 or later
- Maven 3.x

### Running the Application

1.  **Clone the repository:**
    ```sh
    git clone https://github.com/FatimaZahraLasfar/Use-Case-JPA-Hibernate-Spring-Data-Many-To-Many-Case.git
    cd Use-Case-JPA-Hibernate-Spring-Data-Many-To-Many-Case
    ```

2.  **Run the application using the Maven wrapper:**
    ```sh
    ./mvnw spring-boot:run
    ```
    The application will start on `http://localhost:8080`.

### Accessing the H2 Database Console

The project is configured with an in-memory H2 database, and the H2 console is enabled for easy inspection of the data and schema.

1.  Navigate to `http://localhost:8080/h2-console` in your web browser.
2.  Use the following settings to connect:
    - **Driver Class**: `org.h2.Driver`
    - **JDBC URL**: `jdbc:h2:mem:testdb`
    - **User Name**: `sa`
    - **Password**: (leave blank)

Once connected, you can run SQL queries to inspect the `USERS`, `ROLE`, and the generated join table (e.g., `ROLE_USERS`).
