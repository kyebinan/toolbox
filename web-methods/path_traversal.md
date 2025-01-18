# What is path traversal ?
Path traversal is also know as directory traversal. These vulnerabilities enable an attacker to read arbitrary files on the server that is running an application.

In some cases, an attacker might be able to write arbitrary files on the server, allowing them to modify application data or behavior, and ultimately take full control of the server.

# Common obstacles to exploiting path traversal vulnerabilities

    - You might be able to use an absolute path from the filesytem root, to directly reference a file without using any traversal sequences.

    - You might be able to use nested traversal sequences, such as "..//" or "....\/". These revert to simple traversal sequences when the inner sequences is strippedhus 

    - In some contexts, such as in a URL path or the filename parameter of a multipart/form-data request, web servers may strip any directory traversal sequences before passing your input to the application. You can sometimes bypass this kind of sanitization by URL encoding, or even double URL encoding, the ../ characters. This results in %2e%2e%2f and %252e%252e%252f respectively. Various non-standard encodings, such as ..%c0%af or ..%ef%bc%8f, may also work.
