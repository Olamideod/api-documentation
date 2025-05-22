# Cloud Storage API Documentation

## Table of Contents
1. [Getting Started](#getting-started)
2. [Authentication](#authentication)
3. [Base URL and Versioning](#base-url-and-versioning)
4. [Common Headers](#common-headers)
5. [Error Handling](#error-handling)
6. [Rate Limiting](#rate-limiting)
7. [API Endpoints](#api-endpoints)
8. [Code Examples](#code-examples)
9. [SDKs and Libraries](#sdks-and-libraries)
10. [Changelog](#changelog)

## Getting Started

The Cloud Storage API provides programmatic access to manage files, folders, and storage resources in the cloud. This RESTful API supports JSON data format and uses standard HTTP methods.

### Prerequisites
- Valid API credentials (API Key and Secret)
- Basic understanding of HTTP/REST APIs
- JSON knowledge for request/response handling

## Authentication

### API Key Authentication

All API requests require authentication using API Key and Secret pair. Include these in the request headers or use OAuth 2.0 flow for enhanced security.

#### Header Authentication
```http
Authorization: Bearer YOUR_API_KEY
X-API-Secret: YOUR_API_SECRET
```

### OAuth 2.0 Flow

For applications requiring user consent, use the OAuth 2.0 authorization code flow.

#### Step 1: Authorization Request
```http
GET /oauth/authorize?response_type=code&client_id=YOUR_CLIENT_ID&redirect_uri=YOUR_REDIRECT_URI&scope=read,write
```

#### Step 2: Exchange Code for Token
```http
POST /oauth/token
Content-Type: application/json

{
  "grant_type": "authorization_code",
  "client_id": "YOUR_CLIENT_ID",
  "client_secret": "YOUR_CLIENT_SECRET",
  "code": "AUTHORIZATION_CODE",
  "redirect_uri": "YOUR_REDIRECT_URI"
}
```

#### Response
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "def50200..."
}
```

## Base URL and Versioning

- **Base URL**: `https://api.cloudstorage.com`
- **Current Version**: `v1`
- **Full Base URL**: `https://api.cloudstorage.com/v1`

## Common Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | API authentication token |
| `Content-Type` | Yes* | `application/json` for POST/PUT requests |
| `Accept` | No | `application/json` (default) |
| `X-API-Version` | No | API version override |
| `X-Request-ID` | No | Unique request identifier for debugging |

## Error Handling

The API uses conventional HTTP response codes and returns detailed error information in JSON format.

### HTTP Status Codes

| Code | Status | Description |
|------|--------|-------------|
| 200 | OK | Request successful |
| 201 | Created | Resource created successfully |
| 400 | Bad Request | Invalid request parameters |
| 401 | Unauthorized | Invalid or missing authentication |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 409 | Conflict | Resource conflict (e.g., duplicate name) |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |

### Error Response Format

```json
{
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "The filename parameter is required",
    "details": {
      "parameter": "filename",
      "provided": null,
      "expected": "string"
    },
    "request_id": "req_123456789"
  }
}
```

## Rate Limiting

- **Default Limit**: 1000 requests per hour per API key
- **Burst Limit**: 100 requests per minute
- **Headers**: Rate limit information is included in response headers

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 987
X-RateLimit-Reset: 1699123456
```

## API Endpoints

### Files

#### Upload File

Upload a new file to the storage.

```http
POST /files
```

**Parameters:**
- `file` (required): The file to upload
- `path` (optional): Destination path (default: root)
- `overwrite` (optional): Overwrite existing file (default: false)

**Request Example:**
```bash
curl -X POST https://api.cloudstorage.com/v1/files \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "file=@document.pdf" \
  -F "path=/documents/" \
  -F "overwrite=false"
```

**Response:**
```json
{
  "id": "file_abc123",
  "name": "document.pdf",
  "path": "/documents/document.pdf",
  "size": 1048576,
  "mime_type": "application/pdf",
  "created_at": "2024-01-15T10:30:00Z",
  "modified_at": "2024-01-15T10:30:00Z",
  "checksum": "d41d8cd98f00b204e9800998ecf8427e",
  "download_url": "https://cdn.cloudstorage.com/files/file_abc123"
}
```

#### Get File Information

Retrieve metadata for a specific file.

```http
GET /files/{file_id}
```

**Response:**
```json
{
  "id": "file_abc123",
  "name": "document.pdf",
  "path": "/documents/document.pdf",
  "size": 1048576,
  "mime_type": "application/pdf",
  "created_at": "2024-01-15T10:30:00Z",
  "modified_at": "2024-01-15T10:30:00Z",
  "checksum": "d41d8cd98f00b204e9800998ecf8427e",
  "tags": ["important", "contract"],
  "permissions": {
    "public": false,
    "shared": true
  }
}
```

#### Download File

Download a file from storage.

```http
GET /files/{file_id}/download
```

**Parameters:**
- `inline` (optional): Return file inline instead of attachment (default: false)

**Response:**
- Content-Type: Based on file mime type
- Content-Disposition: attachment; filename="document.pdf"
- Raw file content

#### List Files

Retrieve a list of files with optional filtering.

```http
GET /files
```

**Query Parameters:**
- `path` (optional): Filter by path
- `limit` (optional): Number of results (default: 50, max: 200)
- `offset` (optional): Pagination offset (default: 0)
- `sort` (optional): Sort field (name, size, created_at, modified_at)
- `order` (optional): Sort order (asc, desc)
- `mime_type` (optional): Filter by MIME type

**Response:**
```json
{
  "files": [
    {
      "id": "file_abc123",
      "name": "document.pdf",
      "path": "/documents/document.pdf",
      "size": 1048576,
      "mime_type": "application/pdf",
      "created_at": "2024-01-15T10:30:00Z",
      "modified_at": "2024-01-15T10:30:00Z"
    }
  ],
  "pagination": {
    "total": 150,
    "limit": 50,
    "offset": 0,
    "has_more": true
  }
}
```

#### Update File

Update file metadata or replace file content.

```http
PUT /files/{file_id}
```

**Parameters:**
- `name` (optional): New filename
- `path` (optional): New path
- `tags` (optional): Array of tags
- `file` (optional): New file content

**Response:**
```json
{
  "id": "file_abc123",
  "name": "updated_document.pdf",
  "path": "/documents/updated_document.pdf",
  "size": 1048576,
  "mime_type": "application/pdf",
  "created_at": "2024-01-15T10:30:00Z",
  "modified_at": "2024-01-15T14:22:00Z",
  "tags": ["important", "contract", "updated"]
}
```

#### Delete File

Delete a file from storage.

```http
DELETE /files/{file_id}
```

**Response:**
```json
{
  "message": "File deleted successfully",
  "deleted_at": "2024-01-15T15:45:00Z"
}
```

### Folders

#### Create Folder

Create a new folder.

```http
POST /folders
```

**Parameters:**
- `name` (required): Folder name
- `path` (optional): Parent path (default: root)

**Request:**
```json
{
  "name": "Projects",
  "path": "/Work/"
}
```

**Response:**
```json
{
  "id": "folder_xyz789",
  "name": "Projects",
  "path": "/Work/Projects/",
  "created_at": "2024-01-15T16:00:00Z",
  "modified_at": "2024-01-15T16:00:00Z",
  "file_count": 0,
  "folder_count": 0
}
```

#### List Folder Contents

Get contents of a specific folder.

```http
GET /folders/{folder_id}/contents
```

**Query Parameters:**
- `limit` (optional): Number of results (default: 50)
- `offset` (optional): Pagination offset
- `type` (optional): Filter by type (file, folder, all)

**Response:**
```json
{
  "contents": [
    {
      "type": "folder",
      "id": "folder_def456",
      "name": "Subfolder",
      "path": "/Work/Projects/Subfolder/",
      "created_at": "2024-01-15T16:30:00Z"
    },
    {
      "type": "file",
      "id": "file_ghi789",
      "name": "project.docx",
      "path": "/Work/Projects/project.docx",
      "size": 524288,
      "created_at": "2024-01-15T16:45:00Z"
    }
  ],
  "pagination": {
    "total": 25,
    "limit": 50,
    "offset": 0,
    "has_more": false
  }
}
```

### Sharing

#### Create Share Link

Generate a shareable link for a file or folder.

```http
POST /files/{file_id}/share
```

**Parameters:**
- `expires_at` (optional): Expiration timestamp
- `password` (optional): Password protection
- `download_limit` (optional): Maximum downloads

**Request:**
```json
{
  "expires_at": "2024-02-15T00:00:00Z",
  "password": "secret123",
  "download_limit": 10
}
```

**Response:**
```json
{
  "share_id": "share_mno345",
  "share_url": "https://share.cloudstorage.com/s/mno345",
  "expires_at": "2024-02-15T00:00:00Z",
  "download_count": 0,
  "download_limit": 10,
  "created_at": "2024-01-15T17:00:00Z"
}
```

#### Get Share Information

Retrieve information about a share link.

```http
GET /shares/{share_id}
```

**Response:**
```json
{
  "share_id": "share_mno345",
  "file_id": "file_abc123",
  "share_url": "https://share.cloudstorage.com/s/mno345",
  "expires_at": "2024-02-15T00:00:00Z",
  "download_count": 3,
  "download_limit": 10,
  "password_protected": true,
  "created_at": "2024-01-15T17:00:00Z"
}
```

### Account

#### Get Account Information

Retrieve account details and usage statistics.

```http
GET /account
```

**Response:**
```json
{
  "id": "acc_123456",
  "email": "user@example.com",
  "name": "John Doe",
  "plan": "premium",
  "storage": {
    "used": 5368709120,
    "total": 107374182400,
    "used_percentage": 5.0
  },
  "limits": {
    "max_file_size": 5368709120,
    "max_bandwidth": 10737418240,
    "api_requests_per_hour": 5000
  },
  "created_at": "2023-06-01T00:00:00Z"
}
```

## Code Examples

### JavaScript/Node.js

```javascript
const axios = require('axios');

class CloudStorageAPI {
  constructor(apiKey, apiSecret) {
    this.apiKey = apiKey;
    this.apiSecret = apiSecret;
    this.baseURL = 'https://api.cloudstorage.com/v1';
  }

  // Upload a file
  async uploadFile(filePath, destinationPath = '/') {
    const FormData = require('form-data');
    const fs = require('fs');
    
    const form = new FormData();
    form.append('file', fs.createReadStream(filePath));
    form.append('path', destinationPath);

    try {
      const response = await axios.post(`${this.baseURL}/files`, form, {
        headers: {
          ...form.getHeaders(),
          'Authorization': `Bearer ${this.apiKey}`,
          'X-API-Secret': this.apiSecret
        }
      });
      return response.data;
    } catch (error) {
      throw new Error(`Upload failed: ${error.response.data.error.message}`);
    }
  }

  // Get file information
  async getFile(fileId) {
    try {
      const response = await axios.get(`${this.baseURL}/files/${fileId}`, {
        headers: {
          'Authorization': `Bearer ${this.apiKey}`,
          'X-API-Secret': this.apiSecret
        }
      });
      return response.data;
    } catch (error) {
      throw new Error(`Get file failed: ${error.response.data.error.message}`);
    }
  }

  // List files
  async listFiles(options = {}) {
    const params = new URLSearchParams(options);
    
    try {
      const response = await axios.get(`${this.baseURL}/files?${params}`, {
        headers: {
          'Authorization': `Bearer ${this.apiKey}`,
          'X-API-Secret': this.apiSecret
        }
      });
      return response.data;
    } catch (error) {
      throw new Error(`List files failed: ${error.response.data.error.message}`);
    }
  }
}

// Usage
const api = new CloudStorageAPI('your_api_key', 'your_api_secret');

// Upload a file
api.uploadFile('./document.pdf', '/documents/')
  .then(file => console.log('File uploaded:', file))
  .catch(error => console.error(error));

// List files
api.listFiles({ path: '/documents/', limit: 10 })
  .then(result => console.log('Files:', result.files))
  .catch(error => console.error(error));
```

### Python

```python
import requests
import json
from pathlib import Path

class CloudStorageAPI:
    def __init__(self, api_key, api_secret):
        self.api_key = api_key
        self.api_secret = api_secret
        self.base_url = 'https://api.cloudstorage.com/v1'
        self.session = requests.Session()
        self.session.headers.update({
            'Authorization': f'Bearer {api_key}',
            'X-API-Secret': api_secret
        })

    def upload_file(self, file_path, destination_path='/'):
        """Upload a file to cloud storage"""
        url = f'{self.base_url}/files'
        
        with open(file_path, 'rb') as file:
            files = {'file': file}
            data = {'path': destination_path}
            
            response = self.session.post(url, files=files, data=data)
            response.raise_for_status()
            return response.json()

    def get_file(self, file_id):
        """Get file information"""
        url = f'{self.base_url}/files/{file_id}'
        response = self.session.get(url)
        response.raise_for_status()
        return response.json()

    def list_files(self, **kwargs):
        """List files with optional filters"""
        url = f'{self.base_url}/files'
        response = self.session.get(url, params=kwargs)
        response.raise_for_status()
        return response.json()

    def download_file(self, file_id, save_path):
        """Download a file"""
        url = f'{self.base_url}/files/{file_id}/download'
        response = self.session.get(url, stream=True)
        response.raise_for_status()
        
        with open(save_path, 'wb') as file:
            for chunk in response.iter_content(chunk_size=8192):
                file.write(chunk)

    def create_share_link(self, file_id, expires_at=None, password=None):
        """Create a shareable link"""
        url = f'{self.base_url}/files/{file_id}/share'
        data = {}
        if expires_at:
            data['expires_at'] = expires_at
        if password:
            data['password'] = password
            
        response = self.session.post(url, json=data)
        response.raise_for_status()
        return response.json()

# Usage example
if __name__ == '__main__':
    api = CloudStorageAPI('your_api_key', 'your_api_secret')
    
    try:
        # Upload a file
        file_info = api.upload_file('document.pdf', '/documents/')
        print(f"File uploaded: {file_info['name']}")
        
        # List files
        files = api.list_files(path='/documents/', limit=10)
        print(f"Found {len(files['files'])} files")
        
        # Create share link
        share = api.create_share_link(file_info['id'])
        print(f"Share URL: {share['share_url']}")
        
    except requests.exceptions.RequestException as e:
        print(f"API Error: {e}")
```

### cURL Examples

```bash
# Upload a file
curl -X POST https://api.cloudstorage.com/v1/files \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "X-API-Secret: YOUR_API_SECRET" \
  -F "file=@document.pdf" \
  -F "path=/documents/"

# Get file information
curl -X GET https://api.cloudstorage.com/v1/files/file_abc123 \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "X-API-Secret: YOUR_API_SECRET"

# List files with filters
curl -X GET "https://api.cloudstorage.com/v1/files?path=/documents/&limit=10&sort=created_at&order=desc" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "X-API-Secret: YOUR_API_SECRET"

# Download a file
curl -X GET https://api.cloudstorage.com/v1/files/file_abc123/download \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "X-API-Secret: YOUR_API_SECRET" \
  -o downloaded_file.pdf

# Create folder
curl -X POST https://api.cloudstorage.com/v1/folders \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "X-API-Secret: YOUR_API_SECRET" \
  -H "Content-Type: application/json" \
  -d '{"name": "New Folder", "path": "/documents/"}'

# Create share link
curl -X POST https://api.cloudstorage.com/v1/files/file_abc123/share \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "X-API-Secret: YOUR_API_SECRET" \
  -H "Content-Type: application/json" \
  -d '{"expires_at": "2024-02-15T00:00:00Z", "download_limit": 5}'
```

## SDKs and Libraries

### Official SDKs

- **JavaScript/Node.js**: `npm install @cloudstorage/sdk`
- **Python**: `pip install cloudstorage-api`
- **PHP**: `composer require cloudstorage/php-sdk`
- **Go**: `go get github.com/cloudstorage/go-sdk`

### Community Libraries

- **Ruby**: `gem install cloudstorage-ruby`
- **Java**: Available on Maven Central
- **C#**: Available on NuGet

## Webhooks

Configure webhooks to receive real-time notifications about file events.

### Supported Events

- `file.uploaded`
- `file.deleted`
- `file.shared`
- `folder.created`
- `account.quota_exceeded`

### Webhook Configuration

```http
POST /webhooks
```

**Request:**
```json
{
  "url": "https://yourapp.com/webhooks/cloudstorage",
  "events": ["file.uploaded", "file.deleted"],
  "secret": "webhook_secret_key"
}
```

### Webhook Payload Example

```json
{
  "event": "file.uploaded",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "file": {
      "id": "file_abc123",
      "name": "document.pdf",
      "path": "/documents/document.pdf",
      "size": 1048576
    },
    "account_id": "acc_123456"
  }
}
```

## Changelog

### Version 1.2.0 (2024-01-10)
- Added webhook support
- Improved error messages
- Added file checksum validation
- Enhanced search capabilities

### Version 1.1.0 (2023-12-01)
- Added OAuth 2.0 authentication
- Introduced share links with password protection
- Added folder management endpoints
- Improved rate limiting

### Version 1.0.0 (2023-10-15)
- Initial API release
- Basic file operations (upload, download, delete)
- API key authentication
- Account management


