# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a bash-based rclone sync utility for bidirectional Dropbox synchronization. The codebase consists of:

- `rdsync` - Main sync script (git-like workflow for easy use)
- `dropbox-filters.txt` - File exclusion patterns for system/temporary files
- `README.md` - Comprehensive documentation

## Core Architecture

### Main Script (`rdsync`)
- **State Management**: Uses `.rclone-sync-state` file in each local directory to remember remote paths
- **Configuration**: Command-line arguments with sensible defaults, git-like workflow
- **Logging**: Daily logs in `~/.rclone-sync/logs/` directory  
- **Safety Features**: Dry-run mode, automatic first-sync detection, comprehensive error handling

### Key Functions
- `determine_remote_path()` - Handles state persistence and remote path resolution
- `build_rclone_command()` - Constructs rclone bisync command with appropriate flags
- `validate_paths()` - Ensures local/remote accessibility before sync
- `save_state()`/`load_state()` - Manages persistent configuration

### Filter System
- Default exclusions in `.rclone-filters.txt` (per directory) for system files, IDE files, temp files
- Supports custom filter files via `-f` option
- Can be disabled with `--no-exclude-system-files`

## Common Commands

Since this is a bash script project, there are no build/test/lint commands. The script uses a git-like workflow:

```bash
# First sync (from anywhere)
rdsync -l /path/to/local -d RemoteFolder

# OR from within the directory
cd /path/to/local
rdsync -d RemoteFolder

# Subsequent syncs (from within directory)
cd /path/to/local
rdsync

# Dry run
rdsync --dry-run

# Reset state
rdsync --reset-state
```

## Development Notes

### Prerequisites
- `rclone` must be installed and configured with Dropbox remote
- Script requires bash with `set -euo pipefail` for safety

### State File Location
- `.rclone-sync-state` stores remote path configuration
- Located in each local sync directory (per-directory state)
- Contains shell variables: `REMOTE_NAME`, `REMOTE_DIR`, `REMOTE_PATH`, `LAST_SYNC`

### Exit Codes
- `0`: Success
- `1`: Configuration/validation error  
- Other: rclone bisync exit code

### Script Structure
- Lines 1-26: Configuration and setup
- Lines 28-79: Usage/help function
- Lines 81-151: State management functions
- Lines 153-298: Validation and command building
- Lines 300-435: Main execution and argument parsing