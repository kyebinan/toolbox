## API Penetration Testing Cheatsheet

### API Documentation

- **Human-readable Documentation**: Contains detailed explanations, examples, and usage scenarios.
- **Machine-readable Documentation**: Written in structured formats like JSON or XML for automation.

### Discovering API Documentation

- Use Burp Scanner to crawl applications for endpoints like:
  - `/api`
  - `/swagger/index.html`
  - `/openapi.json`

### Interacting with API Endpoints

- **Identifying Endpoints**: Look for patterns in URL structures like `/api/`.
- **Supported HTTP Methods**: Test methods like `GET`, `POST`, `DELETE` to discover functionality.
- **Supported Content Types**: Modify `Content-Type` headers to trigger different behaviors (e.g., JSON vs. XML).

### Using Burp Suite Tools

- **Burp Repeater**: Interact manually with API endpoints.
- **Burp Intruder**: Automate testing by cycling through HTTP methods or payloads.
- **Burp Scanner**: Crawls and audits OpenAPI documentation and endpoints.

### Finding Hidden Endpoints and Parameters

- **Intruder for Hidden Endpoints**: Use wordlists to discover endpoints similar to known paths.
- **Parameter Discovery**: Use tools like Param Miner to guess and test hidden parameters.
- **Content Discovery**: Identify hidden content and parameters not linked from visible browsing.
