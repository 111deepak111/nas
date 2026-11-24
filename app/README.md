# NAS Frontend (Flutter)

Cross-platform client for the custom NAS built with **Flutter**, targeting **Web**, **Android**, and **Desktop** from a single Dart codebase.

## Architecture Overview

The client separates UI presentation from transport and platform-specific storage capabilities:

- **`lib/screens/`**: File browser, media viewer, upload/transfer queue view, and server connection/settings screens.
- **`lib/services/`**:
  - **API & Transfer Client**: Handles authentication tokens, directory listings, file actions (rename, move, delete, create folder), and streaming/multipart uploads.
  - **Platform Storage Abstraction**: Hides differences between browser file APIs, Android storage permissions/media access, and desktop local filesystems using conditional imports or platform adapters.
- **`lib/widgets/`**: Reusable file tiles, breadcrumb navigation, progress indicators, and preview wrappers.

## Key Technical Considerations

### 1. Dynamic Server Targeting & Auth
- Users must be able to configure and switch NAS base URLs (e.g., local LAN IP `http://192.168.1.50:8080`, reverse-proxied HTTPS URLs, or a local server running on the same device).
- Store auth tokens/session credentials securely per platform (avoiding insecure plaintext storage where platform keystores/secure storage are supported).

### 2. Large File Transfers & Media Streaming
- **Streaming Playback**: Play audio/video directly from authenticated or range-enabled stream URLs rather than downloading entire files into memory first.
- **Uploads & Downloads**: Track per-file progress, handle failures/retries cleanly, and avoid blocking the UI thread during large file operations.

### 3. Web vs. Mobile (Android) vs. Desktop Parity
- **Flutter Web**: No raw host filesystem paths; relies on browser upload/download flows and CORS/same-origin alignment when served alongside the Go backend.
- **Android**: Requires runtime storage/media permission handling, attention to background execution limits during long transfers, and clean handling of content URIs/local paths.
- **Desktop**: Supports direct local file picking, drag-and-drop, and standard OS file paths.

## Local Development

```bash
cd app
flutter pub get
flutter run -d chrome      # Web
flutter run -d android     # Android device/emulator
flutter run -d linux       # Or windows / macos