# EOpod: Enhanced TPU Command Runner

EOpod is a command-line tool designed to simplify and enhance interaction with Google Cloud TPU VMs. It provides real-time output streaming, background process management, robust error handling, and support for multiple TPU configurations.

## Features

*   **Multi-Configuration Management:** Easily define and switch between multiple TPU configurations (different projects, zones, or TPU names).
*   **Command Execution:** Run commands on TPU VMs with advanced features like retries, delays, timeouts, and worker selection.
*   **Configuration Logging:** Each command execution logs the active configuration (project, zone, TPU name) for reproducibility.
*   **Command History:** View a history of executed commands, their status, truncated output, and the configuration used.
*   **Error Logging:** Detailed error logs are maintained for debugging failed commands.
*   **Rich Output:** Utilizes the `rich` library for visually appealing and informative output in the console.
*   **TPU Usage Information**: Show current memory usage and duty-cycle for each TPU.

## Installation

```bash
pip install eopod
```

## Configuration

EOpod supports multiple named configurations.  You can create, switch between, and manage different configurations for various projects, zones, or TPU names.

### Creating Configurations

Use the `configure` command with the `--name` option to create a named configuration:

```bash
eopod configure --name my_config --project-id YOUR_PROJECT_ID --zone YOUR_ZONE --tpu-name YOUR_TPU_NAME
```

*   `--name`:  A unique name for your configuration (e.g., `my_tpu`, `project_b`).  If omitted, it defaults to `default`.
*   `--project-id`: Your Google Cloud Project ID.
*   `--zone`: The Google Cloud zone where your TPU is located.
*   `--tpu-name`: The name of your TPU.

**Example:**

```bash
eopod configure --name prod_tpu --project-id my-project --zone us-central1-a --tpu-name my-tpu
eopod configure --name dev_tpu --project-id my-dev-project --zone europe-west4-a --tpu-name dev-tpu-v4-8
```

If you don't provide a `--name`, the configuration will be saved as `default`:

```bash
eopod configure --project-id my-project --zone us-central1-a --tpu-name my-default-tpu
```

### Setting the Active Configuration

The `set-active` command makes a named configuration the default for subsequent `eopod run` commands:

```bash
eopod set-active prod_tpu  # Now, 'eopod run ...' will use the 'prod_tpu' configuration.
```

### Listing Configurations

The `show-config` command displays all your configurations:

```bash
eopod show-config
```

This will show a table of all configurations, indicating which one is currently active.  You can also view a specific configuration:

```bash
eopod show-config --name prod_tpu
```

## Usage Examples

### Basic Command Execution

By default, `eopod run` uses the active configuration (or `default` if none is explicitly set).

```bash
# Uses the active configuration
eopod run echo "Hello TPU"

# Run Python script (using the active configuration)
eopod run python train.py --batch-size 32
```

To use a specific configuration, use the `--config-name` (or `-c`) option:

```bash
eopod run -c dev_tpu python train.py --batch-size 64  # Uses the 'dev_tpu' configuration
```

### Background Processes

```bash
# Start training in background (using the active config)
eopod run python long_training.py --epochs 1000 --background

# Start training using a specific config
eopod run -c prod_tpu python long_training.py --epochs 1000 --background

# Check background processes (shows processes from the active config's TPU)
eopod check-background

# Kill a background process (on the active config's TPU)
eopod kill 12345
```

### Other Commands (Worker Selection, Advanced Options, etc.)

These commands work the same way as before, but they now use the active configuration or the one specified with `--config-name`:

```bash
# Run on specific worker (using the active config)
eopod run nvidia-smi --worker 0

# Run on all workers with a specific config
eopod run -c dev_tpu hostname --worker all

# Set custom retry count (using the active config)
eopod run python train.py --retry 5 --delay 10 --timeout 600
```

### Viewing TPU Usage

```bash
# Install tpu-info (one-time setup)
eopod show-tpu-usage --install-tpuinfo

# Show TPU usage
eopod show-tpu-usage
```

### Viewing History and Logs

The `history` command now also shows which configuration was used for each command:

```bash
# View command history
eopod history

# View error logs
eopod errors
```

## Command Reference

### Main Commands

*   **`run`**: Execute commands on a TPU VM.

    ```bash
    eopod run [OPTIONS] COMMAND [ARGS]...
    ```

    Options:
    *   `--worker TEXT`: Specific worker or "all" (default: "all").
    *   `--retry INTEGER`: Number of retries (default: 1).
    *   `--delay INTEGER`: Delay between retries in seconds (default: 5).
    *   `--timeout INTEGER`: Command timeout in seconds (default: None).
    *   `--no-stream`: Disable output streaming.
    *   `--background`: Run command in the background.
    *   `-c`, `--config-name TEXT`:  Use a specific configuration name (default: "default" or the active configuration).

*   **`configure`**: Create or update a named configuration.

    ```bash
    eopod configure --name NAME --project-id ID --zone ZONE --tpu-name NAME
    ```

*   **`set-active`**: Set the active configuration.

    ```bash
    eopod set-active CONFIG_NAME
    ```

*   **`status`**: Check the status of the TPU (using the active configuration).

    ```bash
    eopod status
    ```

*   **`check-background`**: Check background processes (on the active configuration's TPU).

    ```bash
    eopod check-background [PID]
    ```

*   **`kill`**: Kill background processes (on the active configuration's TPU).

    ```bash
    eopod kill PID [--force]
    ```
 *   **`kill-tpu`**: Kill processes using TPU resources.

      ```bash
      eopod kill-tpu
      ```
      Options:
        *   `--worker TEXT`: Specific worker or "all" (default: "all").
        *   `--force`: Force kill all processes.
        *   `--pid`: Specific PIDs to kill.

### Utility Commands

*   **`history`**: View command execution history (includes configuration used).
*   **`errors`**: View error logs.
*   **`show-config`**: Display all configurations or a specific one.

    ```bash
    eopod show-config  # Show all
    eopod show-config --name my_config  # Show details for 'my_config'
    ```
*    **`show-tpu-usage`**: Show current memory usage and duty-cycle for each TPU.

     ```bash
      eopod show-tpu-usage
      ```
      Options:
        *   `--install-tpuinfo`: installs tpu-info (for first time only).

## File Locations

*   Configuration: `~/.eopod/config.ini`
*   Command history: `~/.eopod/history.yaml`
*   Error logs: `~/.eopod/error_log.yaml`
*   Application logs: `~/.eopod/eopod.log`

## Error Handling

EOpod includes built-in error handling and retry mechanisms:

*   Automatic retry for failed commands.
*   Timeout handling.
*   Detailed error logging.
*   Rich error output.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the Apache License 2.0 - see the LICENSE file for details.
