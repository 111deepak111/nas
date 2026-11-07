# Cross-Platform Network Attached Storage (NAS) System

A unified, containerized Network Attached Storage solution designed to provide secure remote file management, media streaming, and multi-platform access across desktop, mobile, and server environments.

---

## System Overview

The system is split into three core modules:
1. **Core Storage & Server Engine** (Backend APIs & Data Persistence)
2. **Access Interfaces** (Web Browser Interface & Native Mobile/Desktop Applications)
3. **Containerized Deployment Engine** (Cross-platform execution runtime)

---

## Part 1: Server Engine Architecture

The server acts as the centralized coordinator for all storage, authentication, and system communication operations.

### Functional Details
* **File Management:** Supports upload, download, file creation, renaming, moving, and deletion.
* **Streaming Services:** Provides continuous data chunking for media streaming (video, audio, image viewing).
* **Identity & Access Management:** User authentication, password hashing, token management, and role-based access control (RBAC) to enforce permission boundaries on directories.
* **Storage Monitoring:** Reports real-time disk health, available capacity, drive statistics, and overall system usage metrics.

### Technical Details
* **API Interface:** Exposes RESTful endpoints for CRUD file operations and system management, along with WebSocket connections for real-time file transfer progress, drive status updates, and notifications.
* **I/O Subsystem:** Interacts directly with the underlying operating system file handlers using asynchronous stream pipelines to ensure low memory consumption during large file transfers.
* **Security & Auth:** Implements stateless session tokens (JSON Web Tokens) or session cookies alongside encrypted transport (TLS/HTTPS).
* **Metadata Store:** Operates a embedded database or structured file registry to manage user profiles, metadata, system preferences, and file access history.

---

## Part 2: Access Points (User Interfaces)

Users interact with the NAS engine through dual interfaces designed for seamless browser access and dedicated client hardware.

### Functional Details
* **Directory Navigator:** Visual file browser supporting list and grid views, breadcrumbs, search filters, and quick file previews.
* **Transfer Manager:** Multi-file drag-and-drop support, upload/download pause and resume capabilities, and active transfer progress indicators.
* **System Dashboard:** Admin interface for user account management, folder permissions mapping, host performance monitoring, and network configuration.

### Technical Details
* **Browser Access Point:**
  * Single-Page Application (SPA) architecture loading static assets directly served by the backend or a light web server.
  * Direct consumption of backend REST APIs and WebSocket events for state synchronization without full page reloads.
  * Responsive design optimized across screen resolutions.
* **Application Access Point (Mobile & Desktop):**
  * Native binary or unified cross-platform client capable of running on Android, iOS, Windows, macOS, and Linux.
  * Employs persistent network sockets or background sync threads to process transfers even when minimized.
  * Integrates with OS-level APIs for device-native file dialogs, background notifications, and auto-syncing specified system directories to the NAS host.

---

## Part 3: Containerization & Deployment

To ensure consistent behavior across heterogeneous host systems, the NAS backend is packaged as an isolated runtime container.

### Functional Details
* **Portable Execution:** Bootstraps the server environment on any platform with minimal software dependencies.
* **Data Persistence:** Isolates application code from actual storage by mapping physical host drives into the container workspace.
* **Environment Configuration:** Exposes runtime options through simple environment variable files (port binding, storage directory paths, security keys).

### Technical Details
* **Multi-Architecture Support:** Built with cross-platform container images targeting both standard standard 64-bit architectures (`amd64`) and ARM-based hardware (`arm64`).
* **Volume Mapping:** Uses explicit mount points to connect physical server host disks into the container filesystem while maintaining read/write permission mappings.
* **Network Isolation:** Exposes defined host ports to route internal API traffic cleanly while preventing unintended port access on the host machine.

---

## Cross-Platform Compatibility & OS Execution

The containerized backend is designed to deploy across diverse host environments, each with specific operating system and virtualization requirements:

### Linux
* **Execution:** Direct execution via standard container engines.
* **Permissions:** Native access to kernel features (cgroups, namespaces, mount flags) with high performance and zero overhead.

### Windows & macOS
* **Execution:** Runs containerized workloads through background virtualized hypervisors (WSL2 on Windows, Hypervisor framework on macOS).
* **Permissions:** Requires administrator/elevated rights during initial container runtime installation to configure virtual networking adapters and host drive mounts.

### Android
Running the backend on an Android hardware host depends on how the server runtime is launched:

* **Native Binary Execution (No Root Required):**
  * The backend application can be compiled directly for ARM64 and executed inside user-space terminal emulators or embedded directly within a standard Android application APK.
  * Operates within standard Android storage scope privileges without requiring deep kernel access.
* **Containerized Execution (Root Required for Native Engine):**
  * Native container engines rely on Linux kernel features (cgroups, overlay filesystems, network bridging) that stock Android restricts.
  * Running a container engine natively on an Android device requires root access to load custom kernel modules.
  * **Non-Root Alternative:** Running a container on unrooted Android requires virtualizing a lightweight Linux virtual machine first (e.g., via QEMU inside terminal tools), inside of which the container engine runs.