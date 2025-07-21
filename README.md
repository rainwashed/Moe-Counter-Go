# MoeCounter - Moe Style Counter - English Translation (translated using AI)

<div align="center">
  <img src="public/assets/img/Preview.svg" alt="SVG Preview" width="500" style="margin-bottom: 20px;">
<p style="font-size: 18px; color: #555; margin-top: 20px; line-height: 1.6; font-weight: 500;">
    A modular counter service built with Go, using a clear layered architecture (routing/controller/data layers), and offering flexible API interfaces. Supports SVG counter image generation with dozens of built-in themes customizable via parameters.
  </p>
</div>

## Project Structure

```
.
├── cmd/               # CLI entry point
│   ├── root.go        # Root command
│   └── start.go       # Start command
├── database/          # Database module
│   └── sqlite3.go     # SQLite operations
├── public/            # Frontend resources
│   ├── assets/        # Static assets
│   │   └── theme/     # Theme directories
│   ├── favicon.ico    # Site favicon
│   └── index.html     # Homepage
├── server/            # Backend server
│   ├── controller/    # Controllers
│   │   ├── common.go  # Common definitions
│   │   ├── counter.go # Counter logic
│   │   └── theme.go   # Theme logic
│   └── router.go      # Route definitions
├── utils/             # Utility functions
│   └── combine.go     # Combine helpers
├── go.mod             # Go module definition
├── go.sum             # Dependency checksums
└── main.go            # Main program entry
```

## Runtime Logic

1. **Initialization**:

   - Parses CLI arguments (port, database path)
   - Initializes SQLite database
   - Loads embedded static resources

2. **Request Handling**:

   - `/`: Returns homepage HTML
   - `/assets/*`: Serves static assets
   - `/api/counter`: Handles counter requests
   - `/api/themes`: Returns available theme list

3. **Counter Logic**:
   - Retrieves or creates a counter using `name`
   - Increments the counter (unless `num` is specified)
   - Applies `base` offset
   - Returns an SVG image response

## Installation & Running

### Requirements

- Go 1.16+

### Build & Run

```bash
# Build the project
go build -o moeCounter

# Start the service (default: port 8088, DB: data.db)
./moeCounter start

# Custom parameters
./moeCounter start -p 8080 -d custom.db

# Note: Not recommended to run directly in terminal
# Use supervisor tools:
# - 1Panel: Create a daemon process
# - BT Panel: Go project / process manager
# - Others: Use nohup or create a service manually
```

### CLI Parameters

| Parameter | Alias | Default | Description           |
| --------- | ----- | ------- | --------------------- |
| `--port`  | `-p`  | 8088    | Port to listen on     |
| `--db`    | `-d`  | data.db | Path to database file |
| `--debug` | None  | false   | Enable debug mode     |

## API Interface

### Counter Endpoint

`GET /api/counter?name=[counter_name]&[params]`

**Parameters**:

| Parameter  | Type   | Default | Description                            |
| ---------- | ------ | ------- | -------------------------------------- |
| `name`     | string | —       | Counter name (required)                |
| `theme`    | string | random  | Theme name                             |
| `length`   | int    | 7       | Number length                          |
| `scale`    | float  | 1.0     | Image scale                            |
| `offset`   | int    | 0       | Spacing between digits                 |
| `align`    | string | left    | Alignment (left/center/right)          |
| `pixelate` | string | off     | Pixelate effect (on/off)               |
| `darkmode` | string | off     | Dark mode (on/off)                     |
| `base`     | int    | —       | Base offset added to count             |
| `num`      | string | —       | Manually set the number (no increment) |

**Response**: SVG image (`Content-Type: image/svg+xml`)

### Theme List Endpoint

`GET /api/themes`

**Response**:

```json
{
  "themes": ["theme1", "theme2", "theme3"]
}
```

## Theme System

Themes are located in the `public/assets/theme` directory. Each theme is a folder containing digit images (0-9 in .png or .gif format).

Built-in themes include:

- 3d-num
- asoul
- booru series
- capoo
- miku
- minecraft
- and many more

## Examples

```bash
# Basic usage
http://localhost:8088/counter?name=test

# Custom theme and style
http://localhost:8088/counter?name=test&theme=miku&length=5&scale=0.8&align=center

# Set counter value directly
http://localhost:8088/counter?name=test&num=12345
```

## Developer Guide

### Add a New Theme

1. Create a new folder under `public/assets/theme`
2. Add digit images (0-9) in .png or .gif format
3. Optionally add `start` (prefix) and `end` (suffix) images
4. Restart the service to apply changes

### Build & Deploy

```bash
# Build for Linux
GOOS=linux GOARCH=amd64 go build -trimpath --ldflags="-s -w" -o moeCounter

# Build for Windows
GOOS=windows GOARCH=amd64 go build -trimpath --ldflags="-s -w" -o moeCounter.exe

# Build for macOS
GOOS=darwin GOARCH=arm64 go build -trimpath --ldflags="-s -w" -o moeCounter
```

### Docker Requirements

- Create a mount for the /data (internal) directory
- Port must bind to 8088 (internal)
- Docker image can be found here:

  [![rainwashed/moe-counter-go](/readme_assets/docker_btn.svg)](https://hub.docker.com/r/rainwashed/moe-counter-go)

## GitHub Actions Auto Release

We provide a GitHub Actions workflow to automatically build and publish releases when a version tag is pushed.

**How to Use**:

1. Push project code to a GitHub repository
2. Create and push a version tag:
   ```bash
   git tag v1.0.0  # Replace with your version
   git push origin v1.0.0
   ```
3. Visit the "Releases" section in your GitHub repo to view the auto-generated release

**Workflow Overview**:

- Triggered by pushing a `v*.*.*` version tag
- Builds for multiple platforms (Linux/macOS/Windows)
- Automatically uploads all binaries to a release
- Go version: 1.22

**Permissions Setup**:

1. **Default Settings**:

   - GitHub auto-generates `GITHUB_TOKEN` for each workflow
   - No need to manually create/configure it
   - Scope: read/write access to the current repository

2. **Verify and Set Permissions**:

   - Go to `Settings > Actions > General` in your repo
   - Under `Workflow permissions`:
     - Select `Read and write permissions`
     - Check `Allow GitHub Actions to create and approve pull requests`
   - Save the settings

3. **If Permission Denied (403)**:

   ```bash
   # 1. Create a Personal Access Token (PAT) with higher permission:
   #    - Visit https://github.com/settings/tokens
   #    - Click "Generate new token"
   #    - Select "repo" scope
   #    - Copy the generated token

   # 2. Add it to repository Secrets:
   #    - Go to Settings > Secrets > Actions
   #    - Click "New repository secret"
   #    - Name: RELEASE_TOKEN
   #    - Secret: [Paste the PAT]

   # 3. In workflow, use this token:
   env:
     GITHUB_TOKEN: ${{ secrets.RELEASE_TOKEN }}
   ```

4. **Workflow Permission Check**:
   - Every run uses `GITHUB_TOKEN`
   - Check permission level in the job log under "Set up job"
   - If you see `Permission: write`, it's correctly set

## Dependencies

- Gin (web framework)
- GORM (ORM library)
- Cobra (CLI framework)

## Preview

![Preview](public/assets/img/Preview.jpg)

## Credits

Some resources in this project are based on [journey-ad/Moe-Counter](https://github.com/journey-ad/Moe-Counter)

## Star History

<a href="https://star-history.com/?repos=skyle1995/Moe-Counter-Go&type=Date#skyle1995/Moe-Counter-Go&Date">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=skyle1995/Moe-Counter-Go&type=Date&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=skyle1995/Moe-Counter-Go&type=Date" />
   <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=skyle1995/Moe-Counter-Go&type=Date" />
 </picture>
</a>
