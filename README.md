# Quiz Management API

A REST API for managing quiz questions and quizzes. The application is built with Spring Boot, Spring Data JPA, and PostgreSQL. It supports question CRUD operations, creates quizzes from random questions in a category, returns quiz questions without the answer key, and grades submitted responses.

## Technology

- Java 27 (as configured in `pom.xml`)
- Spring Boot 4.1.2
- Spring Web MVC
- Spring Data JPA / Hibernate
- PostgreSQL
- Maven Wrapper
- Lombok

## Requirements

- JDK 27
- PostgreSQL

## Database setup

Create a PostgreSQL database named `questionsDB`. The default connection settings are in `src/main/resources/application.properties`:

| Setting | Default |
| --- | --- |
| URL | `jdbc:postgresql://localhost:5432/questionsDB` |
| Username | `postgres` |
| Password | `postgres` |
| Port | `8081` |

Update these values for your local database before running the app. Hibernate is configured with `spring.jpa.hibernate.ddl-auto=update`, so it updates the schema on startup; it does not populate initial question data.

## Run the application

From the project root, start the application with the Maven Wrapper:

```bash
# Windows
mvnw.cmd spring-boot:run

# macOS / Linux
./mvnw spring-boot:run
```

The API listens on `http://localhost:8081` by default.

To build a package:

```bash
# Windows
mvnw.cmd clean package

# macOS / Linux
./mvnw clean package
```

## API endpoints

All endpoints are relative to `http://localhost:8081`.

### Questions

| Method | Path | Description |
| --- | --- | --- |
| `GET` | `/questions/allQuestions` | List all questions |
| `GET` | `/questions/category/{category}` | List questions in a category |
| `POST` | `/questions/add` | Add a question; returns `201 Created` |
| `PUT` | `/questions/update/{id}` | Replace a question; returns `404` if the ID does not exist |
| `DELETE` | `/questions/delete/{id}` | Delete a question; returns `204 No Content`, or `404` if missing |

Question JSON fields:

```json
{
  "questionTitle": "Which planet is known as the Red Planet?",
  "option1": "Venus",
  "option2": "Mars",
  "option3": "Jupiter",
  "option4": "Mercury",
  "rightAnswer": "Mars",
  "difficultyLevel": "Easy",
  "category": "Science"
}
```

The question list and CRUD responses include `rightAnswer`, so these endpoints should be treated as administrative unless that answer exposure is intended.

### Quizzes

| Method | Path | Description |
| --- | --- | --- |
| `POST` | `/quiz/create?category={category}&numQ={count}&title={title}` | Create a quiz using up to the requested number of random questions in the category; returns `201 Created` and `success` |
| `GET` | `/quiz/get/{id}` | Get the quiz questions and options (without answer keys) |
| `POST` | `/quiz/submit/{id}` | Submit answers and get the number correct as an integer |

Example quiz creation:

```http
POST http://localhost:8081/quiz/create?category=Science&numQ=5&title=Science%20Quiz
```

Example response body for `GET /quiz/get/1`:

```json
[
  {
    "id": 12,
    "questionTitle": "Which planet is known as the Red Planet?",
    "option1": "Venus",
    "option2": "Mars",
    "option3": "Jupiter",
    "option4": "Mercury"
  }
]
```

Submit a JSON array of responses. Each response uses a question `id` and the selected answer text in `response`:

```http
POST http://localhost:8081/quiz/submit/1
Content-Type: application/json
```

```json
[
  { "id": 12, "response": "Mars" }
]
```

The response body is the number of correct answers, for example `1`. Submit responses in the same order as the questions returned by `/quiz/get/{id}`; scoring compares each submitted response with the corresponding quiz question's `rightAnswer`.

## Project layout

```text
src/main/java/com/project/quiz_management/
├── controller/  # REST endpoints
├── dao/         # Spring Data JPA repositories
├── model/       # Question, Quiz, Response, and QuestionWrapper
└── service/     # Question and quiz business logic
src/main/resources/
└── application.properties
```

## Tests

Run the test suite with:

```bash
mvnw.cmd test   # Windows
./mvnw test     # macOS / Linux
```
