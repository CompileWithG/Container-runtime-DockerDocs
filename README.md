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
