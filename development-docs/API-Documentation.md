# RESTful API Reference Blueprint

> **AI Instruction:** Scan all active route files, controllers, and router configurations. Extract every public endpoint and document it using the explicit block layout detailed below.

## 1. Service Context & Base URL

- **Local Base Path:** `http://localhost:5000/api/v1`
- **Global Content-Type:** `application/json`

---

## 2. Endpoint Registry

### [Endpoint Title Example: User Registration]

* **URL String:** `[e.g., /auth/register]`
* **HTTP Protocol Method:** `[POST / GET / PUT / DELETE]`
* **Required Header Keys:** `[e.g., Content-Type: application/json]`
* **Request Payload Payload Schema:**

* **Response:**

  Success (201 Created)

  ```json
    {
      "message": "User registered successfully",
      "userId": "12345"
    }
  ```

  Global Error (400 Bad Request)

  ```json
  {
    "success": false,
    "error": "ERROR_CODE_STRING",
    "message": "Human readable explanation of what failed."
  }
  ```
