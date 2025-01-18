# Path Traversal Vulnerabilities

Path Traversal, also known as Directory Traversal, is a vulnerability that enables an attacker to read arbitrary files on the server running an application.

In some cases, an attacker may even be able to write arbitrary files on the server, modifying application data or behavior, and ultimately gaining full control of the server.

## Common Obstacles to Exploiting Path Traversal Vulnerabilities

- **Absolute Paths**: You may be able to use an absolute path starting from the filesystem root to directly reference a file without the need for traversal sequences.

- **Nested Traversal Sequences**: Attackers may use sequences like `..//` or `....\/` for traversal. In some cases, the inner sequences are stripped, reverting them to simple traversal sequences.

- **Sanitization by Web Servers**: In certain contexts, such as a URL path or a filename parameter in a multipart/form-data request, web servers may strip any directory traversal sequences before passing input to the application. However, you can sometimes bypass this by:
  
  - **URL Encoding**: Encoding the `../` characters as `%2e%2e%2f`.
  - **Double URL Encoding**: Using double URL encoding for the traversal sequences, like `%252e%252e%252f`.
  - **Non-standard Encodings**: Using encodings such as `..%c0%af` or `..%ef%bc%8f` to evade detection.

- **Required Base Folder in Filenames**: Some applications require the user-supplied filename to start with a specific base folder, such as `/var/www/images`. In such cases, it may still be possible to bypass restrictions by including the required base folder followed by suitable traversal sequences. For example:
  
  ```plaintext
  filename=/var/www/images/../../../etc/passwd
  ```

- **Required File Extension**: An application may require the user-supplied filename to end with an expected file extension, such as `.png`. In this case, it might be possible to use a null byte to effectively terminate the file path before the required extension. For example:
  
  ```plaintext
  filename=../../../etc/passwd%00.png
  ```

