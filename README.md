# Complete API Documentation - Assets & Folders Management

## Table of Contents
1. [Overview](#overview)
2. [Authentication](#authentication)
3. [Base URL](#base-url)
4. [Common Concepts](#common-concepts)
5. [Folder Management API](#folder-management-api)
6. [Assets API (Media & Templates)](#assets-api-media--templates)
7. [Error Codes](#error-codes)
8. [Complete Workflow Examples](#complete-workflow-examples)

---

## Overview

This API provides comprehensive folder and asset management capabilities:
- **Folders**: Hierarchical folder structure with parent-child relationships
- **Media Assets**: Images, videos, documents organized in folders
- **Templates**: Design templates with canvas data organized in folders

### Key Features
✅ Hierarchical folder organization with unlimited nesting  
✅ Public/Private folder visibility controls  
✅ Folder-based asset organization  
✅ Duplicate detection with smart copy numbering  
✅ Owner-based authorization  
✅ Signed URLs for secure file access  
✅ Bulk operations support  

---

## Authentication

All endpoints require Bearer token authentication.

```http
Authorization: Bearer your_admin_token_here
```

---

## Base URL

**Development**: `http://localhost:5053`

**API Endpoints**:
- Folders: `/v1/folder/*`
- Assets: `/api/assets/*`

---

## Common Concepts

### Folder Structure
Folders follow a hierarchical structure with parent-child relationships:
```
Root Folder (parent_id: null)
  └── Child Folder 1 (parent_id: root_id)
      └── Grandchild Folder (parent_id: child1_id)
  └── Child Folder 2 (parent_id: root_id)
```

### Folder Properties
- **id**: Unique folder identifier
- **name**: Folder display name
- **description**: Optional description
- **parent_id**: Parent folder ID (null for root folders)
- **is_public**: Visibility flag (true/false)
- **is_active**: Active status (true/false)
- **type**: Folder type (e.g., "panel")
- **fk_org_user_id**: Owner user ID

### Asset Storage in Folders
- Files are stored with `folder_id` embedded in database
- Database format: `"7:filename.png"` (folder_id:filename)
- API returns clean names with separate `folder_id` field
- S3 path: `v1/media/{org_id}/{folder_id}/{filename}`

---

## Folder Management API

### 1. Create Root Folder

**Endpoint**: `POST /v1/folder/new`

**Authorization**: Bearer Token Required

**Request Body**:
```json
{
    "name": "My New Folder",
    "description": "Assets for project X",
    "is_public": false,
    "is_active": true,
    "type": "panel",
    "user_id": 2
}
```

**Field Descriptions**:
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | ✅ Yes | Folder name |
| `description` | string | No | Folder description |
| `is_public` | boolean | No | Public visibility (default: false) |
| `is_active` | boolean | No | Active status (default: true) |
| `type` | string | ✅ Yes | Folder type (e.g., "panel") |
| `user_id` | number | ✅ Yes | Owner user ID |

**Success Response (201)**:
```json
{
    "success": true,
    "data": {
        "id": 7,
        "type": "panel",
        "parent_id": null,
        "fk_org_user_id": 2,
        "name": "My New Folder",
        "description": "Assets for project X",
        "is_active": true,
        "is_public": false,
        "created_at": "2025-09-30T18:37:42.514Z",
        "updated_at": "2025-09-30T18:37:42.514Z"
    }
}
```

**Error Response (400)**:
```json
{
    "success": false,
    "message": "Missing required fields"
}
```

---

### 2. Create Child Folder

**Endpoint**: `POST /v1/folder/new`

**Request Body**:
```json
{
    "name": "Sub Folder",
    "description": "Nested folder",
    "is_public": false,
    "parent_id": 7,
    "is_active": true,
    "type": "panel",
    "user_id": 2
}
```

**Additional Field**:
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `parent_id` | number | ✅ Yes | Parent folder ID for nesting |

**Success Response (201)**:
```json
{
    "success": true,
    "data": {
        "id": 8,
        "type": "panel",
        "parent_id": 7,
        "fk_org_user_id": 2,
        "name": "Sub Folder",
        "description": "Nested folder",
        "is_active": true,
        "is_public": false,
        "created_at": "2025-09-30T18:37:50.939Z",
        "updated_at": "2025-09-30T18:37:50.939Z"
    }
}
```

---

### 3. Get Folder by ID

**Endpoint**: `GET /v1/folder/:folder_id`

**Path Parameters**:
- `folder_id`: Folder ID

**Success Response (200)**:
```json
{
    "success": true,
    "data": {
        "id": 7,
        "type": "panel",
        "parent_id": null,
        "fk_org_user_id": 2,
        "name": "My New Folder",
        "description": "Assets for project X",
        "is_active": true,
        "is_public": false,
        "created_at": "2025-09-30T18:37:42.514Z",
        "updated_at": "2025-09-30T18:37:42.514Z"
    }
}
```

**Error Response (404)**:
```json
{
    "success": false,
    "message": "Folder not found"
}
```

---

### 4. Get All Folders by Organization

**Endpoint**: `GET /v1/folder/org/:org_id`

**Path Parameters**:
- `org_id`: Organization ID

**Success Response (200)**:
```json
{
    "success": true,
    "data": [
        {
            "id": 8,
            "type": "panel",
            "parent_id": 7,
            "fk_org_user_id": 2,
            "name": "Sub Folder",
            "description": "Nested folder",
            "is_active": true,
            "is_public": false,
            "created_at": "2025-09-30T18:37:50.939Z",
            "updated_at": "2025-09-30T18:37:50.939Z"
        },
        {
            "id": 7,
            "type": "panel",
            "parent_id": null,
            "fk_org_user_id": 2,
            "name": "My New Folder",
            "description": "Assets for project X",
            "is_active": true,
            "is_public": false,
            "created_at": "2025-09-30T18:37:42.514Z",
            "updated_at": "2025-09-30T18:37:42.514Z"
        }
    ]
}
```

---

### 5. Get All Public Folders

**Endpoint**: `GET /v1/folder/public/:org_id`

**Path Parameters**:
- `org_id`: Organization ID

Returns only folders where `is_public: true`

**Success Response (200)**:
```json
{
    "success": true,
    "data": [
        {
            "id": 5,
            "type": "panel",
            "parent_id": null,
            "fk_org_user_id": 2,
            "name": "Public Folder",
            "description": "Shared assets",
            "is_active": true,
            "is_public": true,
            "created_at": "2025-09-30T18:20:58.664Z",
            "updated_at": "2025-09-30T18:20:58.664Z"
        }
    ]
}
```

---

### 6. Update Folder

**Endpoint**: `PUT /v1/folder/update-folder`

**Request Body**:
```json
{
    "org_user_id": 2,
    "folder_id": 7,
    "folder_data": {
        "name": "Updated Folder Name",
        "description": "Updated description",
        "is_public": true,
        "is_active": true,
        "type": "panel"
    }
}
```

**Field Descriptions**:
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `org_user_id` | number | ✅ Yes | User making the update |
| `folder_id` | number | ✅ Yes | Folder ID to update |
| `folder_data` | object | ✅ Yes | Updated folder properties |
| `folder_data.name` | string | No | New folder name |
| `folder_data.description` | string | No | New description |
| `folder_data.is_public` | boolean | No | Update visibility |
| `folder_data.is_active` | boolean | No | Update active status |
| `folder_data.type` | string | No | Update folder type |

**Success Response (200)**:
```json
{
    "success": true,
    "data": {
        "id": 7,
        "type": "panel",
        "parent_id": null,
        "fk_org_user_id": 2,
        "name": "Updated Folder Name",
        "description": "Updated description",
        "is_active": true,
        "is_public": true,
        "created_at": "2025-09-30T18:37:42.514Z",
        "updated_at": "2025-09-30T18:45:00.000Z"
    }
}
```

**Error Response (404)**:
```json
{
    "success": false,
    "message": "Folder not found"
}
```

---

### 7. Delete Single Folder

**Endpoint**: `DELETE /v1/folder/delete`

**Request Body**:
```json
{
    "org_user_id": 2,
    "folder_id": 7
}
```

**Field Descriptions**:
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `org_user_id` | number | ✅ Yes | User requesting deletion |
| `folder_id` | number | ✅ Yes | Folder ID to delete |

**Success Response (200)**:
```json
{
    "success": true,
    "message": "Folder deleted successfully"
}
```

**Error Response (404)**:
```json
{
    "success": false,
    "message": "Folder not found"
}
```

**Important Notes**:
- Deleting a parent folder may affect child folders
- Ensure all assets in the folder are handled before deletion
- Consider moving assets to another folder before deleting

---

### 8. Delete Multiple Folders

**Endpoint**: `DELETE /v1/folder/delete/multiple`

**Request Body**:
```json
{
    "org_user_id": 2,
    "folder_ids": [5, 6, 7]
}
```

**Field Descriptions**:
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `org_user_id` | number | ✅ Yes | User requesting deletion |
| `folder_ids` | array | ✅ Yes | Array of folder IDs to delete |

**Success Response (200)**:
```json
{
    "success": true,
    "message": "Folders deleted successfully",
    "deleted_count": 3
}
```

**Partial Success (200)**:
```json
{
    "success": true,
    "message": "Some folders deleted",
    "deleted_count": 2,
    "failed": [
        {
            "folder_id": 7,
            "reason": "Folder not found"
        }
    ]
}
```

**Error Response (404)**:
```json
{
    "success": false,
    "message": "No matching folders found to delete"
}
```

---

## Assets API (Media & Templates)

### Asset Upload Flow
1. **Create/Select Folder** → Get `folder_id`
2. **Upload Asset** → Provide `folder_id`, `org_id`, `org_user_id`
3. **Asset Stored** → Database: `"7:filename.png"`, S3: `v1/media/2/7/filename.png`
4. **Retrieve Assets** → Query by `folder_id` or `org_id`

### Common Asset Fields
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `org_id` | string | ✅ Yes | Organization ID |
| `org_user_id` | string | ✅ Yes | User ID |
| `folder_id` | string | ✅ Yes | Destination folder ID |
| `type` | string | ✅ Yes | "media" or "template" |
| `copy` | boolean | No | Upload as copy if exists (default: false) |

---

### Media Assets

#### 1. Upload Single Media

**Endpoint**: `POST /api/assets/upload-one`

**Content-Type**: `multipart/form-data`

**Request Body** (form-data):
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file` | file | ✅ Yes | File to upload (max 20MB) |
| `org_id` | string | ✅ Yes | Organization ID |
| `org_user_id` | string | ✅ Yes | User ID |
| `folder_id` | string | ✅ Yes | Folder ID |
| `type` | string | ✅ Yes | Must be "media" |
| `copy` | boolean | No | Upload as copy (default: false) |
| `name` | string | No | Custom display name |
| `imageForAll` | boolean | No | Available for all (default: true) |

**Success Response (201)**:
```json
{
    "success": true,
    "data": {
        "id": 22,
        "userId": "2",
        "name": "7:Screenshot 2025-10-01 at 12.51.14 AM-copy.png",
        "fileName": "v1/media/2/7/Screenshot_2025-10-01_at_12.51.14_AM-copy.png",
        "url": "https://gemstone-dev-filestore-digital-assets.s3.us-east-2.amazonaws.com/v1/media/2/7/Screenshot_2025-10-01_at_12.51.14_AM-copy.png",
        "thumbnailKey": "v1/media/2/7/Screenshot_2025-10-01_at_12.51.14_AM-copy.png",
        "thumbnailUrl": null,
        "mimeType": "image/png",
        "size": 14445,
        "width": null,
        "height": null,
        "imageForAll": true,
        "design_element_id": null,
        "type": "media",
        "createdAt": "2025-10-01T10:16:50.111Z",
        "updatedAt": "2025-10-01T10:16:50.113Z"
    },
    "info": "File uploaded as copy"
}
```

**Error Response (409)** - File exists:
```json
{
    "success": false,
    "message": "File already exists",
    "code": "FILE_EXISTS",
    "data": {
        "filename": "screenshot.png",
        "suggestion": "Set 'copy: true' to upload another copy"
    }
}
```

---

#### 2. Get Media by Folder

**Endpoint**: `GET /api/assets/folder`

**Request Body** (JSON):
```json
{
    "folder_id": "7",
    "org_id": "2",
    "org_user_id": "2",
    "type": "media"
}
```

**Success Response (200)**:
```json
{
    "success": true,
    "data": [
        {
            "id": 22,
            "name": "Screenshot 2025-10-01 at 12.51.14 AM-copy.png",
            "folder_id": 7,
            "org_id": "2",
            "size": 14445,
            "mimeType": "image/png",
            "previewUrl": "https://...signed-url...",
            "type": "media"
        }
    ]
}
```

---

#### 3. Get All Media by Organization

**Endpoint**: `GET /api/assets/org/`

**Request Body** (JSON):
```json
{
    "type": "media",
    "org_user_id": "2",
    "org_id": "2"
}
```

**Success Response (200)**:
```json
{
    "success": true,
    "data": [
        {
            "id": 23,
            "name": "Screenshot 2025-09-19 at 9.53.00 PM (2)-copy.png",
            "folder_id": 7,
            "org_id": "2",
            "size": 161831,
            "mimeType": "image/png",
            "previewUrl": "https://...signed-url...",
            "type": "media"
        },
        {
            "id": 22,
            "name": "Screenshot 2025-10-01 at 12.51.14 AM-copy.png",
            "folder_id": 7,
            "org_id": "2",
            "size": 14445,
            "mimeType": "image/png",
            "previewUrl": "https://...signed-url...",
            "type": "media"
        }
    ]
}
```

---

#### 4. Delete Single Media

**Endpoint**: `DELETE /api/assets/item`

**Request Body** (JSON):
```json
{
    "media_id": 22,
    "org_id": "2",
    "org_user_id": "2",
    "type": "media"
}
```

**Success Response (200)**:
```json
{
    "success": true,
    "message": "Deleted",
    "data": {
        "id": 22
    }
}
```

**Error Response (403)** - Not owner:
```json
{
    "success": false,
    "message": "Forbidden: Only the user who uploaded this file can delete it"
}
```

---

#### 5. Delete Multiple Media

**Endpoint**: `DELETE /api/assets/selected`

**Request Body** (JSON):
```json
{
    "ids": [23, 24],
    "org_id": "2",
    "org_user_id": "2",
    "type": "media"
}
```

**Success Response (200)**:
```json
{
    "success": true,
    "data": {
        "deleted": 2,
        "results": [
            {
                "id": 23,
                "status": "deleted"
            },
            {
                "id": 24,
                "status": "deleted"
            }
        ]
    }
}
```

---

### Template Assets

#### 1. Upload Single Template

**Endpoint**: `POST /api/assets/upload-one`

**Content-Type**: `multipart/form-data`

**Request Body** (form-data):
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file` | file | ✅ Yes | Template preview image |
| `org_id` | string | ✅ Yes | Organization ID |
| `org_user_id` | string | ✅ Yes | User ID |
| `folder_id` | string | ✅ Yes | Folder ID |
| `type` | string | ✅ Yes | Must be "template" |
| `templateCanvasData` | string | ✅ Yes | Canvas JSON string |
| `width` | number | ✅ Yes | Template width (px) |
| `height` | number | ✅ Yes | Template height (px) |
| `copy` | boolean | No | Upload as copy |
| `templateName` | string | No | Custom name |
| `templateCategory` | string | No | Category (e.g., "social-media") |

**Example Canvas Data**:
```json
{
    "version": "5.3.0",
    "objects": [
        {
            "type": "rect",
            "width": 100,
            "height": 100,
            "fill": "red"
        }
    ]
}
```

**Success Response (201)**:
```json
{
    "success": true,
    "data": {
        "id": 1,
        "userId": 2,
        "templateName": "7:Screenshot 2025-09-19 at 9.53.00 PM (2)-copy.png",
        "templateCanvasData": "{\"version\":\"5.3.0\",\"objects\":[{\"type\":\"rect\",\"width\":100,\"height\":100,\"fill\":\"red\"}]}",
        "width": 1920,
        "height": 1080,
        "templateImageUrl": "https://gemstone-dev-filestore-digital-assets.s3.us-east-2.amazonaws.com/v1/template/2/7/Screenshot_2025-09-19_at_9.53.00_PM__2_-copy.png",
        "templateCategory": "social-media",
        "createdAt": "2025-10-01T10:29:22.688Z",
        "updatedAt": "2025-10-01T10:29:22.688Z"
    },
    "info": "File uploaded as copy"
}
```

---

## Error Codes

| Status Code | Description | Common Causes |
|-------------|-------------|---------------|
| `200` | OK | Successful operation |
| `201` | Created | Successful creation |
| `400` | Bad Request | Missing fields, invalid data |
| `401` | Unauthorized | Missing/invalid auth token |
| `403` | Forbidden | Not owner, insufficient permissions |
| `404` | Not Found | Resource doesn't exist |
| `409` | Conflict | File already exists |
| `500` | Internal Server Error | Server-side error |

---

## Complete Workflow Examples

### Workflow 1: Create Folder and Upload Media

**Step 1: Create a folder**
```bash
curl -X POST http://localhost:5053/v1/folder/new \
  -H "Authorization: Bearer your_token" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Project Assets",
    "description": "All project files",
    "is_public": false,
    "is_active": true,
    "type": "panel",
    "user_id": 2
  }'

# Response: { "success": true, "data": { "id": 7, ... } }
```

**Step 2: Upload media to the folder**
```bash
curl -X POST http://localhost:5053/api/assets/upload-one \
  -H "Authorization: Bearer your_token" \
  -F "file=@image.png" \
  -F "org_id=2" \
  -F "org_user_id=2" \
  -F "folder_id=7" \
  -F "type=media"

# Response: { "success": true, "data": { "id": 22, ... } }
```

**Step 3: Get all files in the folder**
```bash
curl -X GET http://localhost:5053/api/assets/folder \
  -H "Authorization: Bearer your_token" \
  -H "Content-Type: application/json" \
  -d '{
    "folder_id": "7",
    "org_id": "2",
    "org_user_id": "2",
    "type": "media"
  }'
```

---

### Workflow 2: Organize with Nested Folders

**Step 1: Create root folder**
```bash
POST /v1/folder/new
{
  "name": "Marketing",
  "user_id": 2,
  "type": "panel"
}
# Returns: folder_id = 10
```

**Step 2: Create child folders**
```bash
POST /v1/folder/new
{
  "name": "Social Media",
  "parent_id": 10,
  "user_id": 2,
  "type": "panel"
}
# Returns: folder_id = 11

POST /v1/folder/new
{
  "name": "Email Campaigns",
  "parent_id": 10,
  "user_id": 2,
  "type": "panel"
}
# Returns: folder_id = 12
```

**Step 3: Upload assets to specific folders**
```bash
# Upload to Social Media folder
POST /api/assets/upload-one
form-data: {
  folder_id: 11,
  org_id: 2,
  org_user_id: 2,
  type: "media",
  file: instagram-post.png
}

# Upload to Email Campaigns folder
POST /api/assets/upload-one
form-data: {
  folder_id: 12,
  org_id: 2,
  org_user_id: 2,
  type: "template",
  file: email-template.png,
  templateCanvasData: "{...}",
  width: 600,
  height: 800
}
```

---

### Workflow 3: Handle Duplicate Files

```bash
# First upload
POST /api/assets/upload-one
{ folder_id: 7, file: "logo.png", copy: false }
# Response: 201 Created

# Second upload (same file, no copy)
POST /api/assets/upload-one
{ folder_id: 7, file: "logo.png", copy: false }
# Response: 409 Conflict - "File already exists"

# Third upload (with copy flag)
POST /api/assets/upload-one
{ folder_id: 7, file: "logo.png", copy: true }
# Response: 201 Created - "logo-copy.png"

# Fourth upload (with copy flag)
POST /api/assets/upload-one
{ folder_id: 7, file: "logo.png", copy: true }
# Response: 201 Created - "logo-copy-2.png"
```

---

### Workflow 4: Bulk Operations

**Upload multiple files**
```bash
POST /api/assets/upload-many
form-data: {
  folder_id: 7,
  org_id: 2,
  org_user_id: 2,
  type: "media",
  file: [image1.png, image2.png, image3.png]
}
```

**Delete multiple files**
```bash
DELETE /api/assets/selected
{
  "ids": [20, 21, 22],
  "org_id": "2",
  "org_user_id": "2",
  "type": "media"
}
```

**Delete multiple folders**
```bash
DELETE /v1/folder/delete/multiple
{
  "org_user_id": 2,
  "folder_ids": [5, 6, 7]
}
```

---

## Best Practices

### Folder Management
1. **Create logical hierarchy** - Organize folders by project, department, or content type
2. **Use descriptive names** - Make folders easy to identify
3. **Set appropriate visibility** - Use `is_public` for shared resources
4. **Keep folders active** - Set `is_active: false` for archived folders instead of deleting

### Asset Management
1. **Always specify folder_id** - Organize assets from upload
2. **Handle duplicates intentionally** - Use `copy: true` when needed
3. **Use preview URLs carefully** - They expire after 1 hour
4. **Clean up regularly** - Delete unused assets to save storage

### Security
1. **Verify ownership** - Only owners can modify/delete
2. **Use proper authentication** - Always include valid Bearer token
3. **Validate folder access** - Check folder ownership before operations

---

## Troubleshooting

### Issue: "Folder not found"
**Solution**: Verify folder_id exists using `GET /v1/folder/:folder_id`

### Issue: "File already exists"
**Solution**: Set `copy: true` or delete existing file first

### Issue: "Forbidden: not owner"
**Solution**: Ensure org_user_id matches the uploader's ID

### Issue: "No matching folders found to delete"
**Solution**: Verify folder IDs exist and belong to the organization

### Issue: Preview URL not working
**Solution**: Signed URLs expire after 1 hour - fetch resource again

---

## Support

**API Version**: 1.0.0  
**Last Updated**: October 2025  

For issues:
1. Check this documentation
2. Review error messages
3. Test with Postman collection
