# Blazor StreamBlobVideo

A .NET 8 Blazor Server application demonstrating video streaming capabilities from Azure Blob Storage with multiple implementation approaches.

## Features

- **Dual Video Streaming Methods**
  - SAS Token-based streaming (direct client-to-Azure connection)
  - API proxy streaming (server-mediated streaming)
- **Video Upload Management**
  - Upload MP4 videos with progress tracking
  - Delete existing videos
  - File size validation (500MB limit)
- **Modern UI**
  - Responsive Bootstrap design
  - Real-time updates with SignalR
  - Interactive server components

## Prerequisites

- .NET 8.0 SDK or later
- Azure Storage Account with a blob container
- Visual Studio 2022 or VS Code with C# extensions

## Configuration

1. Update `appsettings.json` with your Azure Storage credentials:

```json
{
  "AzureStorageSettings": {
    "BLOB_CONNECTION_STRING": "your-azure-storage-connection-string",
    "BLOB_CONTAINER_NAME": "videos"
  }
}
```

**Security Note**: For production, use Azure Key Vault or User Secrets instead of storing connection strings in appsettings.json.

## Installation

1. Clone the repository:
```bash
git clone https://github.com/danielmares32/Blazor_StreamBlobVideo_art.git
cd Blazor_StreamBlobVideo_art
```

2. Restore dependencies:
```bash
dotnet restore
```

3. Run the application:
```bash
cd Blazor_VideoStreamBlob
dotnet run
```

4. Navigate to `https://localhost:7121` or `http://localhost:5149`

## Project Structure

```
Blazor_VideoStreamBlob/
├── Components/
│   ├── Layout/
│   │   ├── MainLayout.razor       # Main application layout
│   │   └── NavMenu.razor          # Navigation menu
│   └── Pages/
│       ├── BlobStreamer.razor     # SAS token streaming page
│       ├── BlobStraming.razor     # API proxy streaming page
│       └── VideoUpload.razor      # Video upload management page
├── Infrastructure/
│   └── BLOBConnect.cs             # Azure Blob Storage service
├── wwwroot/                       # Static assets
├── Program.cs                      # Application configuration
└── appsettings.json               # Configuration settings
```

## Usage

### Viewing Videos

1. **SAS Token Streaming** (`/blobstreamer`)
   - Navigate to E-Learner page
   - Select a video from the list
   - Video streams directly from Azure using temporary SAS token
   - Token expires after 15 minutes for security

2. **API Proxy Streaming** (`/blobstreaming`)
   - Navigate to E-Learner Streaming page
   - Select a video from the list
   - Video streams through the server via `/api/getblob/{blobName}` endpoint
   - Better for logging and access control

### Uploading Videos

1. Navigate to Upload Video page (`/videoupload`)
2. Click "Choose File" and select an MP4 video (max 500MB)
3. Click "Upload Video" to start upload with progress tracking
4. View uploaded videos in the list on the right
5. Delete unwanted videos using the delete button

## API Endpoints

- `GET /api/getblob/{blobName}` - Streams video file through server proxy

## Technologies Used

- **.NET 8.0** - Latest .NET framework
- **Blazor Server** - Server-side rendering with real-time updates
- **Azure Blob Storage** - Cloud storage for video files (see [Azure Blob Library Guide](AZURE_BLOB_LIBRARY.md))
- **Bootstrap 5** - Responsive UI framework
- **SignalR** - Real-time client-server communication

## Architecture Highlights

### Streaming Approaches Comparison

| Feature | SAS Token Streaming | API Proxy Streaming |
|---------|-------------------|-------------------|
| **Performance** | High (direct connection) | Lower (server overhead) |
| **Bandwidth Usage** | Minimal server bandwidth | High server bandwidth |
| **Security** | Time-limited tokens | Full server control |
| **Logging** | Limited | Complete request logging |
| **Caching** | Browser/CDN caching | Server-side caching possible |

### Security Features

- Private blob container (no public access)
- SAS tokens with 15-minute expiration
- Read-only permissions for streaming
- File type validation for uploads

## Development

### Building for Production

```bash
dotnet publish -c Release -o ./publish
```

### Running Tests

```bash
dotnet test
```

## Performance Considerations

- **Singleton Service**: BLOBConnect service reuses connections
- **Async Operations**: All I/O operations are asynchronous
- **Stream Processing**: Direct stream forwarding without buffering
- **File Filtering**: Only MP4 files are listed and processed

## Known Limitations

- Maximum upload file size: 500MB
- Only MP4 video format supported
- No video transcoding or optimization
- Basic error handling implementation

## Future Enhancements

- [ ] Add video thumbnail generation
- [ ] Implement video metadata display
- [ ] Add support for multiple video formats
- [ ] Implement user authentication
- [ ] Add video search functionality
- [ ] Implement chunked file uploads for larger files
- [ ] Add video compression options
- [ ] Implement CDN integration for global distribution

## Troubleshooting

### Common Issues

1. **Connection String Errors**
   - Verify Azure Storage connection string in appsettings.json
   - Ensure storage account is accessible

2. **Upload Failures**
   - Check file size (max 500MB)
   - Verify file is MP4 format
   - Ensure blob container exists

3. **Streaming Issues**
   - Check network connectivity
   - Verify blob container permissions
   - Ensure videos are properly uploaded

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- Built with ASP.NET Core Blazor
- Uses Azure Blob Storage for video storage
- Bootstrap for responsive design

## Support

For issues and questions, please create an issue in the [GitHub repository](https://github.com/danielmares32/Blazor_StreamBlobVideo_art/issues).



**Note**: This is a demonstration project for educational purposes. For production use, implement proper security measures including:
- Azure Key Vault for secrets management
- User authentication and authorization
- Comprehensive error handling and logging
- Performance optimization and caching strategies