# Small File Caching Optimization

## Overview

This optimization improves performance when reading small files from Google Cloud Storage by caching the entire file content in memory when it's first accessed. This avoids multiple network requests when reading different parts of the same small file.

## Configuration Options

Two new configuration options have been added:

1. `fs.gs.small.file.cache.enable` (default: false)
   - Enables or disables small file caching
   - When enabled, files smaller than the maximum cache size will be fully fetched and cached in memory

2. `fs.gs.small.file.cache.max.size` (default: 3 * 1024 * 1024 bytes = 3MB)
   - Maximum size in bytes of files that will be cached entirely
   - Files larger than this size will not be cached, even if caching is enabled

## Benefits

- **Reduced Network Requests**: Only one request is needed to fetch the entire file, regardless of how many parts of the file are read
- **Improved Performance**: Subsequent reads from the same file are served from memory, eliminating network latency
- **Better User Experience**: Applications that read small files in multiple chunks will see significant performance improvements

## Implementation Details

The optimization has been implemented in both:
- `GoogleCloudStorageReadChannel` (HTTP API implementation)
- `GoogleCloudStorageClientReadChannel` (Storage Client with gRPC support)

When a file is first accessed, if it's smaller than the configured maximum cache size, the entire file is fetched and stored in memory. Subsequent reads are served directly from the in-memory cache without making additional network requests.

## Usage Example

To enable small file caching in your Hadoop configuration:

```xml
<property>
  <name>fs.gs.small.file.cache.enable</name>
  <value>true</value>
</property>
<property>
  <name>fs.gs.small.file.cache.max.size</name>
  <value>5242880</value> <!-- 5MB -->
</property>
```

## Limitations

- Only works for non-gzipped files
- Memory usage increases with the number of small files being accessed
- Not suitable for very large files (controlled by the max size setting)
