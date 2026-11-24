# NAS Backend (Go)

HTTP/REST and file-streaming backend written in **Go**, responsible for filesystem operations, byte-range media delivery, access control, and serving optional static web assets.

## Architecture Overview

- **`cmd/server/`**: Application entry point, configuration loading (port, storage root, auth secrets, static asset path), signal handling, and graceful shutdown.
- **`internal/storage/`**: Low-level filesystem operations, path resolution, and strict **path-traversal prevention** (`filepath.Clean` + root boundary checks) so requests cannot escape the configured storage root.
- **`internal/files/`**: Directory listing, metadata inspection, streamed uploads (`io.Copy`), file/folder creation, move/rename, deletion, and HTTP `Range` request handling for seeking and resumable streams.
- **`internal/auth/`**: Authentication middleware, session/token validation, and user/role authorization checks on protected routes.

## Key Technical Considerations

### 1. Memory-Safe Streaming & Byte Ranges
- Never buffer entire files into `[]byte` in memory during upload or download.
- Stream request bodies directly to disk and stream files back using `io.Copy` or `http.ServeContent` so `Range` headers work reliably for video/audio scrubbing and partial reads.
- Clean up partial/temporary files if an upload fails or the client disconnects mid-transfer.

### 2. Path Sanitization & Symlink Safety
- Normalize all client-supplied relative paths and verify the resolved target path remains strictly inside the designated storage root directory.
- Be deliberate about symlinks if exposing shared directories so unintended host paths are not leaked.

### 3. Concurrency, Timeouts & Graceful Shutdown
- Configure HTTP server timeouts carefully—note that long-running large file transfers and media streams need appropriate read/write/idle timeout design so active transfers aren't prematurely severed.
- Listen for `SIGINT`/`SIGTERM` to stop accepting new connections and allow in-flight requests to finish cleanly.

### 4. Serving Flutter Web & CORS
- Support serving compiled Flutter Web static files (`build/web`) from the backend when deployed as a single service, falling back to `index.html` for client-side routes where appropriate.
- Configure CORS headers explicitly when running the Flutter Web dev server on a separate origin from the Go API.

## Local Development

```bash
cd server
go mod tidy
go run ./cmd/server