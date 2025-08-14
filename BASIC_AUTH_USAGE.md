# Jira v8.4 Basic Authentication Support

This MCP Server now supports Jira v8.4 basic authentication using the standard HTTP Basic Authentication scheme.

## Overview

Basic authentication allows you to authenticate with Jira Server/Data Center using a username and password (or API token) combination. This is particularly useful for:

- Jira Server/Data Center instances (v8.4+)
- Simple scripts and manual API calls
- Development and testing environments

## How to Use

### 1. HTTP Authorization Header

Send requests with the `Authorization` header using the Basic authentication scheme:

```
Authorization: Basic <base64-encoded-credentials>
```

Where `<base64-encoded-credentials>` is the base64 encoding of `username:password`.

### 2. Example

For a user with:
- Username: `jira_user@company.com`
- Password/API Token: `your_api_token_here`

The credentials string would be: `jira_user@company.com:your_api_token_here`

Base64 encoded: `amlyYV91c2VyQGNvbXBhbnkuY29tOnlvdXJfYXBpX3Rva2VuX2hlcmU=`

Full header: `Authorization: Basic amlyYV91c2VyQGNvbXBhbnkuY29tOnlvdXJfYXBpX3Rva2VuX2hlcmU=`

### 3. Using curl

```bash
# Method 1: Let curl handle the encoding
curl -u "username:password" -H "Content-Type: application/json" \
  http://your-mcp-server/your-endpoint

# Method 2: Manual base64 encoding
curl -H "Authorization: Basic amlyYV91c2VyQGNvbXBhbnkuY29tOnlvdXJfYXBpX3Rva2VuX2hlcmU=" \
  -H "Content-Type: application/json" \
  http://your-mcp-server/your-endpoint
```

### 4. Using Python requests

```python
import requests
import base64

# Method 1: Using requests auth parameter
response = requests.get(
    'http://your-mcp-server/your-endpoint',
    auth=('username', 'password'),
    headers={'Content-Type': 'application/json'}
)

# Method 2: Manual header construction
credentials = base64.b64encode(b'username:password').decode('utf-8')
response = requests.get(
    'http://your-mcp-server/your-endpoint',
    headers={
        'Authorization': f'Basic {credentials}',
        'Content-Type': 'application/json'
    }
)
```

## Security Considerations

⚠️ **Important Security Notes:**

1. **Use HTTPS**: Basic authentication sends credentials in base64 encoding (not encryption). Always use HTTPS in production.

2. **API Tokens**: For Jira Cloud, use API tokens instead of passwords. For Jira Server/Data Center, consider using Personal Access Tokens when available.

3. **Credential Storage**: Never hardcode credentials in your code. Use environment variables or secure credential storage.

4. **CAPTCHA**: Be aware that multiple failed authentication attempts may trigger CAPTCHA, which will prevent API access.

## Supported Authentication Methods

The MCP Server now supports these authentication methods in order of preference:

1. **OAuth 2.0** (most secure, recommended for production)
2. **Personal Access Tokens (PAT)** (good for server/data center)
3. **Basic Authentication** (simple, use with HTTPS)

## Error Handling

The server will return appropriate HTTP 401 errors for:

- Empty or missing Authorization header
- Invalid base64 encoding
- Missing username or password
- Invalid credentials format (missing colon separator)

## Compatibility

- ✅ Jira Server/Data Center v8.4+
- ✅ Jira Cloud (with API tokens)
- ✅ Confluence Server/Data Center
- ✅ Confluence Cloud (with API tokens)

## Migration from Other Auth Methods

If you're currently using environment variables for authentication, you can switch to basic auth headers without changing your server configuration. The basic auth headers will take precedence over environment-based authentication.

## Testing

You can test the basic auth implementation using the provided test scripts:

```bash
python3 test_basic_auth.py
python3 test_basic_auth_logic.py
```

## Troubleshooting

### Common Issues

1. **401 Unauthorized**: Check that your credentials are correct and properly base64 encoded.

2. **Invalid encoding errors**: Ensure your base64 encoding is correct and the credentials contain a colon separator.

3. **CAPTCHA triggered**: If you get authentication denied errors, check if CAPTCHA has been triggered in your Jira instance.

4. **Cloud vs Server**: Make sure you're using the right type of credentials (API tokens for Cloud, username/password or PAT for Server).

### Debug Logging

Enable debug logging to see detailed authentication processing:

```python
import logging
logging.getLogger("mcp-atlassian.servers.main").setLevel(logging.DEBUG)
logging.getLogger("mcp-atlassian.servers.dependencies").setLevel(logging.DEBUG)
```

This will show you exactly how the server is processing your authentication headers.
