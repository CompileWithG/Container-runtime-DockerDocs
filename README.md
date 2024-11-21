This README introduces the fundamentals of container runtimes, namespaces, and cgroups by walking through various examples and tools. Here's a high-level breakdown:

---

### **Core Topics Covered**
1. **Unix `fork` and `exec` Model**:
   - Demonstrates how processes are created using `fork` (to duplicate the process) and `exec` (to run a new program in the process).
   - Example: A program `fork_and_exec` creates a new process and demonstrates parent-child relationships.

2. **Namespaces**:
   - Namespaces provide process isolation by creating separate views of global resources.
   - Types:
     - **PID**: Isolates process IDs.
     - **UTS**: Isolates hostname/domain.
     - **Mount**: Isolates mount points.
     - **Network**: Isolates network interfaces.
     - **IPC**: Isolates inter-process communication.
     - **Cgroup**: Isolates cgroup root directory.
     - **User**: Isolates user/group IDs.
   - Example: Running commands in new namespaces using `ns_child_exec` with flags like `-p` for PID, `-u` for UTS.

3. **Control Groups (cgroups)**:
   - cgroups limit and isolate resource usage like memory, CPU, and IO.
   - Example:
     - Create a memory cgroup (`myapp_mem`), set memory limits, and observe process behavior within these constraints.

4. **Root Filesystem Setup**:
   - Create a root filesystem (rootfs) using `docker export` or utilities like `debootstrap`.
   - Example: Extract an Ubuntu Docker image to create a minimal root filesystem for testing.

5. **`unshare` Command**:
   - Illustrates how to create new namespaces and execute commands in them.
   - Example: Create a new PID namespace and observe process isolation.

---

### **Hands-On Highlights**
- **Building and Running Examples**:
  - Compiling and executing C programs like `fork_and_exec` and `ns_child_exec` to explore process behavior in isolated namespaces.
- **Namespace Testing**:
  - Changing hostnames in isolated UTS namespaces.
  - Verifying PID behavior in new PID namespaces.
- **Cgroups Management**:
  - Limiting memory usage for a process and observing constraints.

---

### **Tools and Commands Used**
- **Compilation**: Using `make` to build C programs.
- **Namespace Utilities**:
  - `ns_child_exec`: Custom tool for running commands in isolated namespaces.
  - `unshare`: Standard Linux tool to isolate resources.
- **cgroups**:
  - `cgcreate`, `cgexec`, `cgdelete` for cgroup management.
- **Filesystem Utilities**:
  - `docker export` to create rootfs.
  - `chroot` to change the root filesystem for a process.

---

### **Takeaways**
- **Namespaces + cgroups = Containers**:
  - The combination of namespaces (to isolate processes) and cgroups (to manage resources) is the foundation of containerization.
- **Hands-On Understanding**:
  - Practical examples like `chroot`, `unshare`, and `cgroup_demo` offer a clear path to understanding how containers are built from first principles.

---
# React App Deployment and Custom Container Environment Setup

This document outlines the steps to deploy a React application and run it in a minimal isolated container-like environment using tools such as `docker`, `unshare`, and `chroot`.

---

## 1. Prepare the React Application

### 1.1. Create a React App
1. **Initialize a New React Project:**
   ```bash
   npx create-react-app my-app
   cd my-app
   ```
   - **What it does:** Sets up a pre-configured React project with the required files and directory structure.

2. **Build the Application:**
   ```bash
   npm run build
   ```
   - **Output:** Creates a `build/` directory containing the production-ready application files.

---

### 1.2. Set Up a Local Web Server
1. **Install `http-server` Globally:**
   ```bash
   npm install -g http-server
   ```

2. **Test the Application Locally:**
   ```bash
   http-server build -p 3000
   ```
   - Open [http://localhost:3000](http://localhost:3000) in a browser to view the app.

---

## 2. Set Up a Root Filesystem

### 2.1. Create a Base Root Filesystem
1. **Start an Ubuntu Container:**
   ```bash
   docker run -it ubuntu:20.04 /bin/bash
   ```
   - **What it does:** Launches an interactive Ubuntu container with a shell.

2. **Install Utilities:**
   ```bash
   apt-get update && apt-get install -y curl net-tools iproute2
   ```
   - **Tools Installed:**
     - `curl`: For fetching resources.
     - `net-tools` & `iproute2`: For networking.

3. **Copy the React App Build Directory to the Container:**
   On the host system:
   ```bash
   docker cp ./my-app/build <container-id>:/opt/build
   ```
   - Ensures the React app files are available inside the container.

4. **Exit the Container:**
   ```bash
   exit
   ```

---

### 2.2. Export and Prepare the Filesystem
1. **Export the Container Filesystem:**
   ```bash
   docker export <container-id> > rootfs.tar
   ```

2. **Extract the Filesystem:**
   ```bash
   mkdir rootfs
   tar -xvf rootfs.tar -C rootfs
   ```
   - **Result:** The `rootfs` directory contains the full container filesystem.

---

## 3. Create and Run a Custom Container

### 3.1. Isolate Namespaces and Chroot
1. **Run the Custom Container:**
   ```bash
   sudo unshare -p -m -u -n -f --mount-proc chroot rootfs /bin/bash
   ```
   - **Flags Explained:**
     - `-p`: Isolates processes.
     - `-m`: Isolates mount points.
     - `-u`: Isolates user IDs.
     - `-n`: Isolates network.
     - `--mount-proc`: Mounts a private `/proc` filesystem.
     - `chroot rootfs`: Changes the root directory to `rootfs`.

   - **What it does:** Starts a minimal isolated environment.

---

### 3.2. Run the React App
1. **Navigate to the Build Directory:**
   ```bash
   cd /opt/build
   ```

2. **Start the Web Server:**
   ```bash
   http-server . -p 3000
   ```

3. **Test the Application:**
   - Access the app in your browser at [http://localhost:3000](http://localhost:3000).

---

## 4. Optional Enhancements

### 4.1. Set Up Networking
1. **Configure a `veth` Pair:**
   - Use `ip link` commands to create a virtual Ethernet connection between the host and the container.

2. **Bridge the Container to the Host Network:**
   - Enable external access by configuring the `veth` interface on the host.

---

### 4.2. Set Resource Limits
1. **Create a cgroup:**
   ```bash
   sudo cgcreate -g memory:my_container
   ```

2. **Set Memory Limits:**
   ```bash
   echo 5242880 > /sys/fs/cgroup/memory/my_container/memory.limit_in_bytes
   ```
   - Restricts the container to 5 MB of memory.

3. **Run the Container in the cgroup:**
   ```bash
   cgexec -g memory:my_container sudo unshare -p -m -u -n -f --mount-proc chroot rootfs /bin/bash
   ```

---

## Key Commands Summary

### Prepare Filesystem:
```bash
docker export <container-id> > rootfs.tar
tar -xvf rootfs.tar -C rootfs
```

### Run the Container:
```bash
sudo unshare -p -m -u -n -f --mount-proc chroot rootfs /bin/bash
```

### Run the React App:
```bash
http-server /opt/build -p 3000
```

---
