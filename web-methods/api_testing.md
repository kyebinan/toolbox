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
- **Description**: Occurs when frameworks bind user inputs to internal object fields, exposing unintended parameters.
- **Testing**: Modify hidden parameters like `isAdmin` to test for privilege escalation.

### 7. Server-side Parameter Pollution (SSPP)
- **Description**: Improperly handled user input enables attackers to modify or inject parameters in server-side requests.
- **Testing**:
  - **Query String Injection**: Modify query strings with special characters (`#`, `&`, `=`).
  - **Overriding Parameters**: Inject parameters with the same name to test handling:
    ```http
    GET /userSearch?name=peter%26name=carlos&back=/home
    ```

### 8. REST Path Parameter Pollution
- **Description**: Manipulate path parameters in RESTful APIs.
- **Testing**: Use URL encoding for path traversal (e.g., `peter/../admin`).

### 9. Testing for Server-side Parameter Pollution in Structured Data Formats
- **Description**: Manipulate parameters in JSON or XML formats.
- **Testing**: Inject unexpected structured data and observe server-side response.
  - **Example**: Adding `access_level` to a user profile update request:
    ```json
    {"name":"peter","access_level":"administrator"}
    ```

### 10. Structured Format Injection
- **Description**: Injection vulnerabilities occur when user input is improperly added to structured formats (e.g., JSON, XML).
- **Testing**: Modify JSON responses or requests to inject malicious parameters, such as adding `access_level` to JSON data.

### 11. Automated Tools for Detection
- **Burp Scanner**: Detect suspicious input transformations.
- **Backslash Powered Scanner**: Identify server-side injection vulnerabilities.
- **Manual Verification**: Investigate inputs flagged as “interesting” or “vulnerable” by automated tools.

### 12. Preventing Server-side Parameter Pollution
- **Prevention**: Use an allowlist to define acceptable characters and ensure encoding of all user inputs before they are included in server-side requests. Enforce input validation for expected formats and structures.

---

This cheatsheet condenses the most important information for penetration testing APIs, including tools and techniques for discovering vulnerabilities, as well as best practices for preventing server-side parameter pollution.
