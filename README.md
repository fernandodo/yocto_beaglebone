# Yocto Beaglebone
A customized yocto project for beaglebone. 

## WSL Setup
Setting up a Yocto Project build environment on a non-startup disk in WSL involves several steps. Here's a detailed guide:

---

### **Step 1: Install WSL**
1. **Enable WSL and Install a Supported Linux Distribution**:
   - Open PowerShell as Administrator and enable WSL:
     ```powershell
     wsl --install
     ```
   - Verify WSL 2 is installed:
     ```powershell
     wsl --set-default-version 2
     ```

2. **Install a Supported Linux Distro**:
   - Yocto supports distributions like Ubuntu (20.04, 22.04) or Debian (11, 12). Install Ubuntu:
     ```powershell
     wsl --install -d Ubuntu-22.04
     ```

---

### **Step 2: Set Up WSL on a Non-Startup Disk**
1. **Create a Directory on the Desired Disk**:
   - Navigate to your non-startup disk (e.g., `D:`) and create a directory for WSL:
     ```powershell
     mkdir E:\WSL
     ```

2. **Export and Import the Linux Distro**:
   - Export the installed distro to a tar file:
     ```powershell
     wsl --export Ubuntu-22.04 E:\WSL\Ubuntu-22.04.tar
     ```
   - Unregister the current WSL installation:
     ```powershell
     wsl --unregister Ubuntu-22.04
     ```
   - Import the distro to the new location:
     ```powershell
     wsl --import Ubuntu-22.04 E:\WSL\Ubuntu-22.04 D:\WSL\Ubuntu-22.04.tar --version 2
     ```

3. **Start the Distro**:
   - Launch the WSL distro:
     ```powershell
     wsl -d Ubuntu-22.04
     ```
   - Set it as the default distro:
     ```powershell
     wsl --set-default Ubuntu-22.04
     ```
     
To set up a normal user as the default user after importing an image in WSL, and to configure their home folder as the default login folder, follow these steps:

#### **Step 1: Create a New User**
1. Launch your WSL instance:
   ```bash
   wsl -d <DistroName>
   ```
   Replace `<DistroName>` with the name of your imported distribution.

2. Create a new user:
   ```bash
   sudo adduser <username>
   ```
   Replace `<username>` with your desired username. Follow the prompts to set a password and additional user details.

3. Add the user to necessary groups (e.g., `sudo`):
   ```bash
   sudo usermod -aG sudo <username>
   ```

#### **Step 2: Set the New User as the Default User**
1. Open the WSL configuration file for your distribution. In your WSL terminal:
   ```bash
   sudo nano /etc/wsl.conf
   ```
2. Add or modify the following lines:
   ```plaintext
   [user]
   default = <username>
   ```
   Replace `<username>` with the name of your new user.

3. Save and exit the file (`Ctrl + O`, `Enter`, `Ctrl + X`).

#### **Step 3: Set the Home Folder as the Default Login Folder**
1. Edit the `.bashrc` file for the new user:
   ```bash
   sudo nano /home/<username>/.bashrc
   ```
   Add the following line at the end of the file:
   ```bash
   cd /home/<username>
   ```
   Replace `<username>` with your username.

#### **Step 4: Apply the Changes**
1. Close and restart your WSL instance:
   ```powershell
   wsl --shutdown
   wsl -d <DistroName>
   ```
2. Verify that the new user is logged in by default and that the home folder is the default directory:
   ```bash
   whoami
   pwd
   ```

#### **Optional: Remove Root Login**
If you want to restrict root login:
1. Open the `/etc/passwd` file:
   ```bash
   sudo nano /etc/passwd
   ```
2. Comment out the root shell configuration (optional step for additional security). For example, change:
   ```plaintext
   root:x:0:0:root:/root:/bin/bash
   ```
   To:
   ```plaintext
   # root:x:0:0:root:/root:/bin/bash
   ```

This setup ensures that your newly created user logs in by default, with their home folder as the login directory. Let me know if you encounter issues!

---

### **Step 3: Update and Prepare the Distro**
1. **Update Package Lists and Install Dependencies**:
   - Update the distro:
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```
   - Install Yocto Project dependencies:
     ```bash
     sudo apt install -y gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping libsdl1.2-dev xterm
     ```

2. **Ensure Locale Settings Are Correct**:
   - Check locale:
     ```bash
     locale
     ```
   - If incorrect, generate and set the desired locale:
     ```bash
     sudo locale-gen en_US.UTF-8
     sudo update-locale LANG=en_US.UTF-8
     export LANG=en_US.UTF-8
     ```

---

### **Step 4: Optimize WSL for Building Yocto**
1. **Configure Memory and Processor Resources**:
   - Edit or create the `.wslconfig` file in your Windows user directory:
     ```plaintext
     [wsl2]
     memory=8GB
     processors=4
     swap=0
     ```
   - Restart WSL to apply the changes:
     ```powershell
     wsl --shutdown
     ```

2. **Mount Non-Startup Disk with Optimal Performance**:
   - Ensure your non-startup disk is mounted with metadata support by editing `/etc/wsl.conf`:
     ```bash
     sudo nano /etc/wsl.conf
     ```
     Add the following:
     ```plaintext
     [automount]
     root = /mnt/
     options = "metadata,umask=22,fmask=11"
     ```
   - Restart WSL.
These lines configure the behavior of WSL when it mounts drives from your Windows system into the Linux environment. Here's what each line does in detail:

#### **Line-by-Line Explanation**
1. **`[automount]`**:
   - This section header indicates that the following settings relate to the automatic mounting of Windows drives in WSL.

2. **`root = /mnt/`**:
   - Specifies the base directory in WSL where Windows drives (e.g., `C:` or `D:`) will be mounted.  
   - By default, drives are mounted under `/mnt/`. For example, the `D:` drive would be accessible as `/mnt/d/`.

3. **`options = "metadata,umask=22,fmask=11"`**:
   - Configures mount options for the drives. Each part defines specific behaviors:
     - **`metadata`**:  
       - Enables support for Linux file metadata (e.g., permissions and ownership) on mounted Windows drives.  
       - Without this, file permissions in WSL are simplified, which can cause issues for tools like `git` and build systems that depend on Unix-like file permissions.
     - **`umask=22`**:  
       - Sets the default permission mask for directories.  
       - A `umask` of `22` corresponds to permissions `755` (read, write, execute for owner; read and execute for group and others). This is typical for directories in Linux.
     - **`fmask=11`**:  
       - Sets the default permission mask for files.  
       - A `fmask` of `11` corresponds to permissions `644` (read and write for owner; read-only for group and others). This is common for files.

#### **Purpose in Yocto or Development Context**
- Ensures Linux-like behavior on mounted Windows drives, crucial for:
  - File permissions and ownership management.
  - Compatibility with tools like `BitBake`, `git`, and compilers.
  - Preventing permission-related errors during Yocto builds, especially on non-startup disks.

#### **Common Use Case**
If you're using a non-startup disk (like `D:`) to store Yocto build directories, this configuration ensures that:
1. The disk is mounted properly in WSL (`/mnt/d/`).
2. Files and directories behave as expected in Linux (permissions-wise).
3. Build systems like Yocto work without encountering errors related to incorrect file metadata.

---

### **Step 5: Set Up the Yocto Environment**
1. **Clone the Yocto Project Repository**:
   - Navigate to your desired directory:
     ```bash
     cd /mnt/d/yocto-build
     mkdir poky
     cd poky
     ```
   - Clone the Poky repository:
     ```bash
     git clone git://git.yoctoproject.org/poky.git
     cd poky
     ```

2. **Checkout the Desired Branch**:
   Check the website: https://www.yoctoproject.org/
   - Example for Yocto Scarthgap:
     ```bash
     git checkout -t origin/scarthgap -b my-scarthgap
     ```

4. **Set Up and Initialize the Build Environment**:
   - Source the build environment setup script:
     ```bash
     source oe-init-build-env
     ```

---

### **Verification**
- Install the Required Tools
  ```bash
  sudo apt install -y lz4 zstd
  ```
  
- Run a minimal build to confirm:
  ```bash
  bitbake core-image-minimal
  ```
If the build completes successfully, your environment is set up and ready.

Let me know if you encounter any issues!
