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

### 9. Testing for server-side parameter pollution in structured data formats 

An attacker may be able to manipulate paramaters to exploit vulnerabilities in the server's processing of other structured data formats, such as a JSON or XML. To test for this, inject unexpected structured data into user inputs and see how the server responds. 
Consider an application that enables users to edit their profile, then applies their changes with a request to a server-side API. When you edit your name, your browser makes the following request :

POST /myaccount
name=peter

This results in the following server-side request:
PATCH /users/7312/update
{"name":"peter"}

You can attempt to add the access_level parameter to the request as follows:
POST /myaccount
name=peter","access_level":"administrator

If the user input is added to the server-side JSON data without adequate validation or sanitization, this results in the following server-side request:
PATCH /users/7312/update
{name="peter","access_level":"administrator"}

This may result in the user peter being given administrator access. 

### 10. Testing for server-side parameter pollution in structured data formats-continued 

Consider a similar example, but where the client-side user input is in JSON data. When you edit your name, your browser makes the following request:

POST /myaccount
{"name": "peter"}

This results in the following server-side request:
PATCH /users/7312/update
{"name":"peter"}

You can attempt to add the access_level parameter to the request as follows:
POST /myaccount
{"name": "peter\",\"access_level\":\"administrator"}

If the user input is decoded, then added to the server-side JSON data without adequate encoding, this results in the following server-side request:
PATCH /users/7312/update
{"name":"peter","access_level":"administrator"}

Again, this may result in the user peter being given administrator access.

Structured format injection can also occur in responses. For example, this can occur if user input is stored securely in a database, then embedded into a JSON response from a back-end API without adequate encoding. You can usually detect and exploit structured format injection in responses in the same way you can in requests.
Note

This example below is in JSON, but server-side parameter pollution can occur in any structured data format. For an example in XML, see the XInclude attacks section in the XML external entity (XXE) injection topic.


### 11. Testing with automated tools 

 Burp includes automated tools that can help you detect server-side parameter pollution vulnerabilities.

Burp Scanner automatically detects suspicious input transformations when performing an audit. These occur when an application receives user input, transforms it in some way, then performs further processing on the result. This behavior doesn't necessarily constitute a vulnerability, so you'll need to do further testing using the manual techniques outlined above. For more information, see the Suspicious input transformation issue definition.

You can also use the Backslash Powered Scanner BApp to identify server-side injection vulnerabilities. The scanner classifies inputs as boring, interesting, or vulnerable. You'll need to investigate interesting inputs using the manual techniques outlined above. For more information, see the Backslash Powered Scanning: hunting unknown vulnerability classes whitepaper. 

### 12. Preventing server-side parameter pollution 

To prevent server-side parameter pollution, use an allowlist to define characters that don't need encoding, and make sure all other user input is encoded before it's included in a server-side request. You should also make sure that all input adheres to the expected format and structure.
