# Rclone Dropbox Bidirectional Sync

A robust bash script for bidirectional synchronization between local directories and Dropbox using rclone's `bisync` command.

## Features

- **True bidirectional sync** using rclone bisync
- **Persistent remote path memory** - remembers the original Dropbox path after first sync
- **Comprehensive CLI options** for all configuration parameters
- **Optional system file filtering** - exclude temporary/system files or sync everything
- **Safety features** - dry-run mode, access checks, comprehensive logging
- **State management** - tracks sync history and remote path configuration

## Prerequisites

1. **Install rclone**: Follow [rclone installation guide](https://rclone.org/install/)
2. **Configure Dropbox remote**: Run `rclone config` and set up your Dropbox remote

## Quick Start

1. **First sync** (establishes the remote path):
   ```bash
   ./rclone-dropbox-sync.sh -l /path/to/local/folder -d MyDropboxFolder --resync
   ```

2. **Regular syncs** (uses remembered remote path):
   ```bash
   ./rclone-dropbox-sync.sh -l /path/to/local/folder
   ```

3. **Move local folder** (still syncs to same remote):
   ```bash
   ./rclone-dropbox-sync.sh -l /different/local/path
   ```

## Command Line Options

### Required
- `-l, --local-dir DIR` - Local directory to sync

### Remote Configuration
- `-r, --remote-name NAME` - Rclone remote name (default: dropbox)
- `-d, --remote-dir DIR` - Remote directory path (default: root)
- `--force-remote-path` - Override remembered remote path
- `--reset-state` - Reset remembered remote path

### Sync Options
- `--dry-run` - Preview changes without making them
- `--exclude-system-files` - Use filters for system/temp files (default: true)
- `--no-exclude-system-files` - Sync everything including system files
- `--resync` - Perform initial resync (required for first run)
- `--resilient` - Handle minor errors without requiring resync

### Advanced Options
- `-f, --filters-file FILE` - Custom filters file
- `--log-dir DIR` - Log directory (default: ./logs)
- `--no-check-access` - Skip access validation
- `--verbose` - Enable verbose output

## Usage Examples

### Initial Setup
```bash
# First time setup with specific Dropbox folder
./rclone-dropbox-sync.sh -l ~/Documents -d Work/Documents --resync

# Sync everything including system files
./rclone-dropbox-sync.sh -l ~/Documents -d Work/Documents --resync --no-exclude-system-files
```

### Regular Usage
```bash
# Regular sync (uses remembered remote path)
./rclone-dropbox-sync.sh -l ~/Documents

# Dry run to see what would happen
./rclone-dropbox-sync.sh -l ~/Documents --dry-run

# Verbose sync with detailed output
./rclone-dropbox-sync.sh -l ~/Documents --verbose
```

### Changing Configuration
```bash
# Change to different remote folder
./rclone-dropbox-sync.sh -l ~/Documents -d DifferentFolder --force-remote-path

# Reset state and start fresh
./rclone-dropbox-sync.sh --reset-state
./rclone-dropbox-sync.sh -l ~/Documents -d NewFolder --resync
```

## How Remote Path Memory Works

1. **First Sync**: You specify the remote path with `-d` and `--resync`
2. **State Storage**: Script saves the remote path in `.sync-state` file
3. **Subsequent Syncs**: Script automatically uses the saved remote path
4. **Location Independence**: You can sync from any local directory to the same remote

### State File Location
The state is stored in `.sync-state` next to the script:
```
rclone-sync/
├── rclone-dropbox-sync.sh
├── .sync-state              # Remembers remote path
├── dropbox-filters.txt
└── logs/
```

## File Filtering

### Default Behavior (--exclude-system-files)
Excludes common system and temporary files:
- Dropbox metadata (.dropbox, .dropbox.cache)
- System files (.DS_Store, Thumbs.db, desktop.ini)
- Version control (.git/, .svn/)
- IDE files (.vscode/, .idea/)
- Temporary files (*.tmp, *.temp, ~*, .swp)
- Build artifacts (node_modules/, __pycache__)

### Sync Everything (--no-exclude-system-files)
Syncs all files including system and temporary files.

### Custom Filters
The script automatically creates `.rclone-filters.txt` in each sync directory on first run. You can:

1. **Use the auto-created file**: Customize `.rclone-filters.txt` in your sync directory
2. **Specify a custom file**: Use `-f` option to specify a different filters file:
   ```bash
   ./rdsync -l ~/Documents -f my-custom-filters.txt
   ```

**Note**: If you modify filter files after bisync is established, you must run `--resync` to update rclone's state.

## Logging

Logs are stored in `logs/` directory with timestamps:
```
logs/
├── rclone-sync-20250125-143022.log
├── rclone-sync-20250125-150133.log
└── ...
```

Each log contains:
- Sync start/end times
- Files transferred
- Errors and warnings
- State management activities

## Error Handling

### Common Issues

1. **"Remote not found"**:
   ```bash
   rclone config  # Configure your Dropbox remote
   ```

2. **"First run requires --resync"**:
   ```bash
   ./rclone-dropbox-sync.sh -l ~/Documents -d MyFolder --resync
   ```

3. **"Local directory does not exist"**:
   Ensure the local path exists before syncing.

4. **"filters file has changed (must run --resync)"**:
   When filter files are modified after bisync is established, rclone requires resync:
   ```bash
   ./rdsync --resync
   ```

### Recovery Options

- Use `--resilient` for handling minor errors
- Use `--resync` to recover from serious issues (will overwrite newer files)
- Check logs in `logs/` directory for detailed error information

## Safety Features

- **Dry Run**: `--dry-run` shows what would happen without making changes
- **Access Checks**: Validates both local and remote paths before sync
- **Comprehensive Logging**: All operations are logged with timestamps
- **State Persistence**: Remembers configuration between runs
- **Filter Support**: Prevents syncing of problematic files

## Integration

### Cron Job Example
```bash
# Sync every hour
0 * * * * /path/to/rclone-sync/rclone-dropbox-sync.sh -l /home/user/Documents
```

### Exit Codes
- `0`: Success
- `1`: Configuration or validation error
- Other: rclone bisync exit code

## Troubleshooting

1. **Check rclone configuration**:
   ```bash
   rclone listremotes
   rclone lsd dropbox:
   ```

2. **Test connectivity**:
   ```bash
   ./rclone-dropbox-sync.sh -l ~/Documents --dry-run --verbose
   ```

3. **Reset if needed**:
   ```bash
   ./rclone-dropbox-sync.sh --reset-state
   ```

## Contributing

The script is designed to be easily customizable. Key areas for modification:
- Filter patterns in `dropbox-filters.txt`
- Default configuration variables at the top of the script
- Logging format in the `log()` function
- Additional rclone options in `build_rclone_command()`