# securin

# Recipe Data — Spring Boot REST API

A Spring Boot application that loads a large recipe dataset from a JSON file into a MySQL database and exposes a REST API to query and manage recipes.

---

## Tech Stack

- **Java 21**
- **Spring Boot 4.0.3**
- **Spring Data JPA** (Hibernate)
- **MySQL** (via `mysql-connector-j`)
- **Lombok** (boilerplate reduction)
- **Jackson** (JSON parsing)
- **Maven** (build tool)

---

## Project Structure

```
Recipe Data/
└── demo/
    ├── pom.xml
    ├── src/
    │   └── main/
    │       ├── java/com/example/demo/
    │       │   ├── DemoApplication.java          # App entry point
    │       │   ├── config/
    │       │   │   ├── Config.java               # Bean configuration
    │       │   │   └── RecipeDataLoader.java      # Auto-loads recipes.json on startup
    │       │   ├── controller/
    │       │   │   └── RecipeController.java      # REST endpoints
    │       │   ├── dto/
    │       │   │   └── RecipeResponseDTO.java     # Response shape
    │       │   ├── model/
    │       │   │   └── Recipe.java               # JPA entity
    │       │   ├── repository/
    │       │   │   └── RecipeRepository.java      # Data access layer
    │       │   └── service/
    │       │       └── RecipeService.java         # Business logic
    │       └── resources/
    │           ├── application.properties         # DB & JPA config
    │           └── recipes.json                   # Source dataset (~17 MB)
    └── target/                                    # Compiled build output
```

---

## Data Model

The `Recipe` entity (mapped to the `recipes` table) contains:

| Field         | Type               | Description                          |
|---------------|--------------------|--------------------------------------|
| `id`          | `Integer`          | Auto-generated primary key           |
| `title`       | `String`           | Recipe name                          |
| `cuisine`     | `String`           | Cuisine type (e.g., Italian, Indian) |
| `rating`      | `Float`            | User rating                          |
| `prep_time`   | `Integer`          | Preparation time (minutes)           |
| `cook_time`   | `Integer`          | Cooking time (minutes)               |
| `total_time`  | `Integer`          | Auto-calculated: prep + cook         |
| `description` | `String (TEXT)`    | Recipe description                   |
| `nutrients`   | `Map<String,String>` | Nutritional info stored as JSON     |
| `serves`      | `String`           | Serving size                         |

---

## API Endpoints

### Create a Recipe
```
POST /recipes
Content-Type: application/json
```
**Body:** Recipe JSON object  
**Response:** `RecipeResponseDTO` of the saved recipe

---

### Get Top Recipes (by Rating)
```
GET /recipes/top?limit=2
```
**Query Param:** `limit` — number of recipes to return (default: `2`)  
**Response:**
```json
{
  "data": [
    {
      "id": 1,
      "title": "...",
      "cuisine": "...",
      "rating": 4.9,
      ...
    }
  ]
}
```

---

## Getting Started

### Prerequisites

- Java 21+
- Maven 3.x
- MySQL running locally

### 1. Configure the Database

Edit `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/recipess
spring.datasource.username=your_username
spring.datasource.password=your_password
```

> **Note:** The current file contains a hardcoded password. Replace it before sharing or deploying.

Create the database in MySQL:
```sql
CREATE DATABASE recipess;
```

### 2. Build and Run

```bash
cd "Recipe Data/demo"
./mvnw spring-boot:run
```

On first startup, `RecipeDataLoader` will automatically parse `recipes.json` and bulk-insert all recipes into the database. Subsequent restarts skip this step to avoid duplicates.

### 3. Test the API

```bash
# Get top 5 recipes by rating
curl "http://localhost:8080/recipes/top?limit=5"
```

---

## Notes

- `spring.jpa.hibernate.ddl-auto=create` will **drop and recreate** the `recipes` table on each startup. Change to `update` or `validate` for production use.
- The `recipes.json` dataset is approximately 17 MB and is bundled inside `src/main/resources/`.
- Batch insert is configured (`batch_size=1000`) for efficient loading of the large dataset.
