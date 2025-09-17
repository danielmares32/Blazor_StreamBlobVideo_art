# Azure Blob Storage Library Overview

This document explains how the Azure Blob Storage library (`Azure.Storage.Blobs`) is used in the Blazor StreamBlobVideo project.

## What is Azure Blob Storage?

Azure Blob Storage is Microsoft's object storage solution for the cloud, optimized for storing massive amounts of unstructured data like videos, images, documents, and backups.

## Library: Azure.Storage.Blobs (v12.22.2)

### Key Components Used

#### 1. **BlobServiceClient**
The main entry point for interacting with Azure Blob Storage.

```csharp
var blobSvcClient = new BlobServiceClient(_connectionString);
```

**Purpose**: Authenticates and provides access to storage account operations.

#### 2. **BlobContainerClient**
Represents a specific container within the storage account.

```csharp
var containerClient = blobSvcClient.GetBlobContainerClient(_containerName);
```

**Purpose**: Manages container-level operations like listing blobs, setting access policies.

#### 3. **BlobClient**
Represents an individual blob (file) within a container.

```csharp
var blobClient = containerClient.GetBlobClient(blobName);
```

**Purpose**: Handles file-specific operations like upload, download, delete, and generating SAS tokens.

## Core Operations in Our Project

### 1. **Listing Blobs** (`ListBlobs()`)

```csharp
var blobs = containerClient.GetBlobs()
    .Select(b => b.Name)
    .Where(b => b.EndsWith(".mp4"))
    .ToList();
```

**What it does**: Retrieves all blob names in the container, filters for MP4 files only.

### 2. **Generating SAS Tokens** (`GetBlobSasToken()`)

```csharp
var sasBuilder = new BlobSasBuilder(BlobContainerSasPermissions.Read, 
    DateTimeOffset.UtcNow.AddMinutes(15))
{
    BlobContainerName = _containerName,
    BlobName = blobName,
    Resource = "b"
};

var sasToken = blobClient.GenerateSasUri(sasBuilder).Query;
```

**What it does**: Creates time-limited URLs (15 minutes) that allow direct browser access to videos without exposing storage keys.

### 3. **Streaming Files** (`GetBlobFileStream()`)

```csharp
return await blobClient.OpenReadAsync();
```

**What it does**: Opens a direct read stream from Azure Blob Storage for server-side streaming.

### 4. **Uploading Files** (`UploadVideoAsync()`)

```csharp
await blobClient.UploadAsync(fileStream, new BlobUploadOptions
{
    HttpHeaders = new BlobHttpHeaders { ContentType = contentType }
});
```

**What it does**: Uploads video files to blob storage with proper MIME type headers.

### 5. **Deleting Files** (`DeleteVideoAsync()`)

```csharp
var response = await blobClient.DeleteIfExistsAsync();
return response.Value;
```

**What it does**: Safely deletes blobs, returns true if deletion was successful.

## Security Features

### **SAS (Shared Access Signature) Tokens**
- **Purpose**: Provide temporary, limited access to resources
- **Permissions**: Read-only access to specific blobs
- **Expiration**: 15-minute time limit for security
- **Benefits**: No need to expose storage account keys to clients

### **Access Control**
```csharp
await containerClient.SetAccessPolicyAsync(PublicAccessType.None);
```
- Container is set to private (no anonymous access)
- All access requires authentication or SAS tokens

## Connection String Configuration

```json
{
  "AzureStorageSettings": {
    "BLOB_CONNECTION_STRING": "DefaultEndpointsProtocol=https;AccountName=...;AccountKey=...;EndpointSuffix=core.windows.net",
    "BLOB_CONTAINER_NAME": "videos"
  }
}
```

**Components**:
- **AccountName**: Your Azure Storage account name
- **AccountKey**: Access key for authentication
- **EndpointSuffix**: Azure cloud endpoint (usually core.windows.net)

## Performance Optimizations

### **Async Operations**
All blob operations use async/await patterns:
```csharp
await blobClient.UploadAsync(fileStream);
await blobClient.OpenReadAsync();
await blobClient.DeleteIfExistsAsync();
```

### **Stream Processing**
Direct stream access without loading entire files into memory:
```csharp
using var stream = await blobClient.OpenReadAsync();
// Stream directly to client without buffering
```

### **Singleton Service**
BLOBConnect is registered as singleton to reuse connections:
```csharp
builder.Services.AddSingleton<BLOBConnect>();
```

## Error Handling

### **Basic Exception Handling**
```csharp
try
{
    await blobClient.UploadAsync(fileStream);
    return true;
}
catch (Exception)
{
    return false;
}
```

### **Existence Checks**
```csharp
await containerClient.CreateIfNotExistsAsync();  // Create container if missing
await blobClient.DeleteIfExistsAsync();          // Delete only if exists
```

## Common Blob Storage Patterns

### **1. Direct Client Access (SAS Tokens)**
```
Client → SAS Token Request → Server
       → Direct Azure Access → Azure Blob Storage
```
**Benefits**: Reduced server load, better performance
**Use case**: Public content, temporary access needed

### **2. Server Proxy Pattern**
```
Client → Server → Azure Blob Storage → Server → Client
```
**Benefits**: Full control, logging, processing capabilities
**Use case**: Private content, access control required

## Additional Features Available

### **Metadata and Properties**
```csharp
var properties = await blobClient.GetPropertiesAsync();
var metadata = properties.Value.Metadata;
```

### **Conditional Operations**
```csharp
await blobClient.UploadAsync(stream, conditions: new BlobRequestConditions
{
    IfNoneMatch = ETag.All  // Only upload if blob doesn't exist
});
```

### **Blob Tiers**
```csharp
await blobClient.SetAccessTierAsync(AccessTier.Cool);  // Cost optimization
```

## Best Practices Applied

1. **Connection Reuse**: Singleton service pattern
2. **Async Operations**: Non-blocking I/O
3. **Security**: Private containers with SAS tokens
4. **Resource Management**: Using statements for streams
5. **Error Handling**: Try-catch blocks for critical operations

## Limitations & Considerations

1. **File Size Limits**: 500MB in our implementation (configurable)
2. **Supported Formats**: MP4 only in current implementation
3. **Error Granularity**: Basic error handling (production needs more detail)
4. **Connection String Security**: Should use Azure Key Vault in production

This Azure Blob Storage integration provides a robust, scalable solution for video storage and streaming in the Blazor application.