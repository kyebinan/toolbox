## API Penetration Testing Cheatsheet

### 1. API Documentation

- **Human-readable Documentation**: Explanations, examples, and usage scenarios.
- **Machine-readable Documentation**: Structured formats like JSON or XML for automation.

### 2. Discovering API Documentation

- **Use Burp Scanner**: Crawl for common endpoints:
  - `/api`
  - `/swagger/index.html`
  - `/openapi.json`

### 3. Interacting with API Endpoints

- **Identifying Endpoints**: Look for patterns like `/api/`.
- **Test HTTP Methods**: Use methods like `GET`, `POST`, `DELETE` to uncover functionality.
- **Test Content Types**: Modify `Content-Type` headers (e.g., JSON vs. XML).

### 4. Burp Suite Tools

- **Burp Repeater**: Manually interact with endpoints.
- **Burp Intruder**: Automate testing (methods, payloads).
- **Burp Scanner**: Crawl and audit OpenAPI documentation.

### 5. Finding Hidden Endpoints and Parameters

- **Use Intruder**: Discover endpoints similar to known paths with wordlists.
- **Parameter Discovery**: Use tools like Param Miner to test hidden parameters.
- **Content Discovery**: Identify hidden content or parameters not linked from visible browsing.

### 6. Mass Assignment Vulnerabilities

- **Description**: Occurs when frameworks automatically bind user inputs to internal object fields, potentially exposing unintended parameters.
- **Testing for Mass Assignment**:
  - **Example**: Test modifying hidden parameters like `isAdmin`:
    ```json
    {
      "username": "wiener",
      "email": "wiener@example.com",
      "isAdmin": true
    }
    ```
  - **Exploit**: If the parameter is bound without validation, it could grant elevated privileges (e.g., admin rights).

### 7. Server-side Parameter Pollution (SSPP)

- **Description**: Occurs when user input is improperly handled, enabling attackers to modify or inject parameters in server-side requests.
- **Testing for SSPP**:
  - **Query String Injection**: Modify query strings with special characters (`#`, `&`, `=`) to manipulate server-side requests:
    ```http
    GET /userSearch?name=peter%23foo&back=/home
    ```
  - **Truncation or Injection**: Add valid or invalid parameters (e.g., `foo=xyz`).
  - **Overriding Parameters**: Inject parameters with the same name to test how the application handles conflicting values:
    ```http
    GET /userSearch?name=peter%26name=carlos&back=/home
    ```

### 8. REST Path Parameter Pollution

- **Description**: Manipulate path parameters to exploit vulnerabilities in RESTful APIs.
- **Testing for Path Parameter Pollution**:
  - **Path Traversal**: Use URL encoding to manipulate path parameters (e.g., `peter/../admin`).
  - **Example**:
    ```http
    GET /edit_profile.php?name=peter%2f..%2fadmin
    ```
  - **Exploit**: If the path is normalized, this could resolve to `/api/private/users/admin`, potentially exposing unauthorized data or actions.

---

This format highlights the key penetration testing techniques with a focus on practical exploitation strategies, improving readability and utility.
