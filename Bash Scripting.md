## Bash Script

Bash scripting is a useful tool for automating operations, system administration, and workflow optimization. The fact that it comes pre-installed in most Unix-like operating systems makes it a must-have skill for developers, system administrators, and power users.

Our bash script will perform thefollowing:

- accept arguments to take action

- if no argument give a help on how the script is used

- if argument **1** is given. check system date and write it a file in **/tmp**

- if argument **2** is given, check system installed packages, users, groups, and store this in separate files within a directories, use script to create directories as may be required

- if argument **3** is passed, - install and configure **kubectl**, **helm** and **terraform** binaries (only latest versions)
  have a logic where:
     -  script gives output of the versions of kubectl, helm and terraform ouput in screen and same information is stored in a file
  
  - capture errors and die if not working
  
  - show debug messages when running script, but give option for use to turn it of

## **Script Structure**

### **1. Shebang and Debugging**

```bash
#!/bin/bash
```

- This line tells the system that this is a **bash script** and should be executed using the **bash shell**.

```bash
if [[ "$DEBUG" == "true" ]]; then
 set -x # Enable debug mode
fi
```

- If the `DEBUG` environment variable is set to `true`, the script will run in **debug mode**, which prints each command before executing it. This is useful for troubleshooting.

---

### **2. Help Function**

```bash
show_help() {
 echo "Usage: $0 [OPTION]"
 echo "Options:"
 echo " 1 Check system date and write it to /tmp/system_date.txt"
 echo " 2 Check installed packages, users, and groups, and store them in separate files"
 echo " 3 Install and configure kubectl, helm, and terraform (latest versions)"
 echo " --help Show this help message"
 exit 1
}
```

- This function displays a **help message** explaining how to use the script. It is called if no arguments are provided or if the `--help` argument is passed.

---

### **3. Error Handling**

```bash
handle_error() {
 echo "Error: $1" >&2
 exit 1
}
```

- This function prints an **error message** and stops the script if something goes wrong.

---

### **4. Argument Handling**

```bash
if [[ $# -eq 0 ]]; then
 show_help
fi
```

If no arguments are provided, the script calls the `show_help` function to display usage instructions.

```bash
case $1 in
 1)
 # Argument 1: Write system date to /tmp/system_date.txt
 echo "Checking system date..."
 date > /tmp/system_date.txt || handle_error "Failed to write system date to file"
 echo "System date written to /tmp/system_date.txt"
 ;;
```

- If the first argument is `1`, the script writes the current system date to a file (`/tmp/system_date.txt`).
  
  2)

```bash
  2)
    # Argument 2: Check installed packages, users, and groups, and store them in separate files
    echo "Checking installed packages, users, and groups..."
    OUTPUT_DIR="/tmp/system_info"
    mkdir -p "$OUTPUT_DIR" || handle_error "Failed to create directory $OUTPUT_DIR"

    # Check installed packages
    dpkg -l > "$OUTPUT_DIR/installed_packages.txt" || handle_error "Failed to list installed packages"

    # Check users
    cut -d: -f1 /etc/passwd > "$OUTPUT_DIR/users.txt" || handle_error "Failed to list users"

    # Check groups
    cut -d: -f1 /etc/group > "$OUTPUT_DIR/groups.txt" || handle_error "Failed to list groups"

    echo "System information stored in $OUTPUT_DIR"
    ;;
```

- If the first argument is `2`, the script:
  
  - Creates a directory (`/tmp/system_info`) to store system information.
  
  - Lists installed packages and saves the output to `installed_packages.txt`.
  
  - Lists users and groups and saves them to `users.txt` and `groups.txt`.
  
  3)
  
  # Argument 3: Install and configure kubectl, helm, and terraform
  
    echo "Installing kubectl, helm, and terraform..."

```bash
# Install kubectl
echo "Installing kubectl..."
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" || handle_error "Failed to download kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/ || handle_error "Failed to move kubectl to /usr/local/bin"

# Install helm
echo "Installing helm..."
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 || handle_error "Failed to download helm installer"
chmod +x get_helm.sh
./get_helm.sh || handle_error "Failed to install helm"
rm get_helm.sh
# Install terraform
echo "Installing terraform..."
TERRAFORM_VERSION=$(curl -s https://api.github.com/repos/hashicorp/terraform/releases/latest | grep tag_name | cut -d '"' -f 4 | sed 's/v//')  # Remove 'v' from version
TERRAFORM_ZIP="terraform_${TERRAFORM_VERSION}_linux_amd64.zip"
TERRAFORM_URL="https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/${TERRAFORM_ZIP}"

# Download Terraform
echo "Downloading Terraform ${TERRAFORM_VERSION}..."
curl -LO "$TERRAFORM_URL" || handle_error "Failed to download Terraform"

# Verify the downloaded file
if ! file "$TERRAFORM_ZIP" | grep -q "Zip archive data"; then
  handle_error "Downloaded Terraform file is not a valid ZIP archive"
fi

# Unzip Terraform
echo "Unzipping Terraform..."
unzip "$TERRAFORM_ZIP" || handle_error "Failed to unzip Terraform"
sudo mv terraform /usr/local/bin/ || handle_error "Failed to move Terraform to /usr/local/bin"
rm "$TERRAFORM_ZIP"

# Display versions and store in a file
echo "Versions installed:"
echo "kubectl: $(kubectl version --client --output=json | jq -r '.clientVersion.gitVersion')"
echo "helm: $(helm version --short)"
echo "terraform: $(terraform version | head -n 1)"

echo "kubectl: $(kubectl version --client --output=json | jq -r '.clientVersion.gitVersion')" > /tmp/versions.txt
echo "helm: $(helm version --short)" >> /tmp/versions.txt
echo "terraform: $(terraform version | head -n 1)" >> /tmp/versions.txt
echo "Installation complete. Version information stored in /tmp/versions.txt"
;;
```

- If the first argument is `3`, the script:
  
  - Installs `kubectl`, `helm`, and `terraform` (latest versions).
  
  - Displays and saves the installed versions to `/tmp/versions.txt`.

```bash
--help)
 show_help
 ;;
 *)
 echo "Invalid option: $1"
 show_help
 ;;
esac
```

- If the argument is `--help`, the script displays the help message.

- If the argument is invalid, the script displays an error message and the help message.

---
##General Script
```bash
#!/bin/bash

# Enable debugging if DEBUG environment variable is set
if [[ "$DEBUG" == "true" ]]; then
  set -x  # Enable debug mode
fi

# Function to display help message
show_help() {
  echo "Usage: $0 [OPTION]"
  echo "Options:"
  echo "  1       Check system date and write it to /tmp/system_date.txt"
  echo "  2       Check installed packages, users, and groups, and store them in separate files"
  echo "  3       Install and configure kubectl, helm, and terraform (latest versions)"
  echo "  --help  Show this help message"
  exit 1
}

# Function to handle errors
handle_error() {
  echo "Error: $1" >&2
  exit 1
}

# Check if no arguments are provided
if [[ $# -eq 0 ]]; then
  show_help
fi

# Process arguments
case $1 in
  1)
    # Argument 1: Write system date to /tmp/system_date.txt
    echo "Checking system date..."
    date > /tmp/system_date.txt || handle_error "Failed to write system date to file"
    echo "System date written to /tmp/system_date.txt"
    ;;
  2)
    # Argument 2: Check installed packages, users, and groups, and store them in separate files
    echo "Checking installed packages, users, and groups..."
    OUTPUT_DIR="/tmp/system_info"
    mkdir -p "$OUTPUT_DIR" || handle_error "Failed to create directory $OUTPUT_DIR"

    # Check installed packages
    dpkg -l > "$OUTPUT_DIR/installed_packages.txt" || handle_error "Failed to list installed packages"

    # Check users
    cut -d: -f1 /etc/passwd > "$OUTPUT_DIR/users.txt" || handle_error "Failed to list users"

    # Check groups
    cut -d: -f1 /etc/group > "$OUTPUT_DIR/groups.txt" || handle_error "Failed to list groups"

    echo "System information stored in $OUTPUT_DIR"
    ;;
  3)
    # Argument 3: Install and configure kubectl, helm, and terraform
    echo "Installing kubectl, helm, and terraform..."

    # Install kubectl
    echo "Installing kubectl..."
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" || handle_error "Failed to download kubectl"
    chmod +x kubectl
    sudo mv kubectl /usr/local/bin/ || handle_error "Failed to move kubectl to /usr/local/bin"

    # Install helm
    echo "Installing helm..."
    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 || handle_error "Failed to download helm installer"
    chmod +x get_helm.sh
    ./get_helm.sh || handle_error "Failed to install helm"
    rm get_helm.sh

    # Install terraform
    echo "Installing terraform..."
    TERRAFORM_VERSION=$(curl -s https://api.github.com/repos/hashicorp/terraform/releases/latest | grep tag_name | cut -d '"' -f 4 | sed 's/v//')  # Remove 'v' from version
    TERRAFORM_ZIP="terraform_${TERRAFORM_VERSION}_linux_amd64.zip"
    TERRAFORM_URL="https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/${TERRAFORM_ZIP}"

    # Download Terraform
    echo "Downloading Terraform ${TERRAFORM_VERSION}..."
    curl -LO "$TERRAFORM_URL" || handle_error "Failed to download Terraform"

    # Verify the downloaded file
    if ! file "$TERRAFORM_ZIP" | grep -q "Zip archive data"; then
      handle_error "Downloaded Terraform file is not a valid ZIP archive"
    fi

    # Unzip Terraform
    echo "Unzipping Terraform..."
    unzip "$TERRAFORM_ZIP" || handle_error "Failed to unzip Terraform"
    sudo mv terraform /usr/local/bin/ || handle_error "Failed to move Terraform to /usr/local/bin"
    rm "$TERRAFORM_ZIP"

    # Display versions and store in a file
    echo "Versions installed:"
    echo "kubectl: $(kubectl version --client --output=json | jq -r '.clientVersion.gitVersion')"
    echo "helm: $(helm version --short)"
    echo "terraform: $(terraform version | head -n 1)"

    echo "kubectl: $(kubectl version --client --output=json | jq -r '.clientVersion.gitVersion')" > /tmp/versions.txt
    echo "helm: $(helm version --short)" >> /tmp/versions.txt
    echo "terraform: $(terraform version | head -n 1)" >> /tmp/versions.txt

    echo "Installation complete. Version information stored in /tmp/versions.txt"
    ;;
  --help)
    show_help
    ;;
  *)
    echo "Invalid option: $1"
    show_help
    ;;
esac
```
## **How to Use the Script**

1. Save the script to a file, e.g., `system_script.sh`.

2. Make it executable:
   
   ```bash
   - chmod +x system_script.sh
   ```
   
   Run the script with an argument:

3. - Check system date:
     
     ```bash
     ./system_script.sh 1
     ```
   
   - Check system information:
     
     ```bash
     ./system_script.sh 2
     ```
   
   - Install tools:
     
     ```bash
     ./system_script.sh 3
     ```
   
   - Show help:
     
     ```bash
     ./system_script.sh --help
     ```

4. Enable debugging (optional):
   
   ```bash
   DEBUG=true ./system_script.sh 3
   ```
   
   ## **Key Features**
   
   - **Error Handling**: The script stops and displays an error message if something goes wrong.
   
   - **Debugging**: You can enable debug mode to see each command before it runs.
   
   - **Flexibility**: The script performs different tasks based on the argument you provide.
