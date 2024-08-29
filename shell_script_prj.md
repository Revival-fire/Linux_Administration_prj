##   Automated Server Update and Web Server Installation Script

 Here's a detailed shell script that incorporates remote server access, system updates, and web server installation, along with parameterization and error handling.

### Shell Script: `deploy.sh`

```bash
#!/bin/bash

# Function to display usage information
usage() {
    echo "Usage: $0 -s \"server1 server2 server3\" [-p ssh_port] [-u username] [-h]"
    echo "  -s  List of server addresses or hostnames (required)"
    echo "  -p  SSH port (optional, default: 22)"
    echo "  -u  SSH username (optional, default: current user)"
    echo "  -h  Display this help message"
    exit 1
}

# Default values
ssh_port=22
ssh_user=$(whoami)

# Parse command-line options
while getopts ":s:p:u:h" opt; do
    case ${opt} in
        s)
            servers=(${OPTARG})
            ;;
        p)
            ssh_port=${OPTARG}
            ;;
        u)
            ssh_user=${OPTARG}
            ;;
        h | *)
            usage
            ;;
    esac
done

# Check if the servers list is provided
if [ -z "${servers}" ]; then
    echo "Error: No servers provided."
    usage
fi

# Define the command to update the system and install Nginx
update_and_install() {
    sudo apt-get update -y && sudo apt-get upgrade -y
    if [ $? -ne 0 ]; then
        echo "System update failed."
        exit 1
    fi

    sudo apt-get install -y nginx
    sudo apt-get install apache2 -y

    if [ $? -ne 0 ]; then
        echo "Nginx and apache installation failed."
        exit 1
    fi
}

# Function to check if a command was successful
check_success() {
    if [ $? -eq 0 ]; then
        echo "Success: $1"
    else
        echo "Failed: $1" >&2
    fi
}

# Loop through each server and execute the update and install commands
for server in "${servers[@]}"; do
    echo "Connecting to $server..."

    # Use SSH to run the update and install command on the remote server
    ssh -p ${ssh_port} -o BatchMode=yes -o ConnectTimeout=10 ${ssh_user}@${server} "$(typeset -f update_and_install); update_and_install"
    
    # Check if the SSH command was successful
    check_success "Update, Nginx and apache  installation on $server"
done

echo "Deployment complete."
```

### Explanation of the script above is below 

1. **Parameterization**:
   - **Server List (`-s`)**: Accepts a space-separated list of server addresses or hostnames.
   - **SSH Port (`-p`)**: Optional, allows the user to specify a custom SSH port (default is 22).
   - **SSH User (`-u`)**: Optional, allows the user to specify a custom SSH username (default is the current user).

2. **Error Handling**:
   - **Usage Information**: Displays a help message if the user provides incorrect options or misses required parameters.
   - **Connection Errors**: Checks for successful SSH connections. If the connection fails, an error message is displayed.
   - **Command Execution**: Checks the success of system updates and Nginx installation on each server. If any command fails, an appropriate error message is displayed.

3. **Execution Logic**:
   - The script loops through each server provided in the `-s` parameter.
   - Uses SSH to remotely run system updates and Nginx installation commands.
   - Verifies the success of each operation and provides feedback to the user.

### Usage

1. **Make the Script Executable**:
   ```bash
   chmod +x deploy_servers.sh
   ```

2. **Run the Script**:
   ```bash
   ./deploy_servers.sh -s "server1.example.com server2.example.com" -p 22 -u username
   ```

   - **`-s`**: (Required) A space-separated list of server addresses or hostnames.
   - **`-p`**: (Optional) SSH port. Default is `22`.
   - **`-u`**: (Optional) SSH username. Default is the current user.

3. **Example**:
   ```bash
   ./deploy_servers.sh -s "192.168.1.10 192.168.1.11" -p 22 -u admin
   ```

### Error Handling

- **Missing Servers**: If no servers are provided, the script will display an error and the usage information.
- **SSH Failures**: If SSH connection fails due to incorrect username, port, or server address, the script will inform the user.
- **Command Failures**: If the system update or Nginx installation fails, the script will notify the user and stop further execution on that server.

