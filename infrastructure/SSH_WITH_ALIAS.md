# SSH Connection with an Alias

This guide shows you how to configure an alias for an SSH connection, replacing the server's IP address with a more user-friendly name.

## Steps

### 1. Edit or Create the SSH Config File

Open or create the SSH config file located at `~/.ssh/config` on your local machine:

```bash
nano ~/.ssh/config
```

### 2. Add a New Entry for Your Server

Add a new entry in the file to define the alias. Here's an example:

```plaintext
Host raspberry
    HostName MY_IP_ADDRESS
    User baco
```

- `Host`: Your alias. In this case, `raspberry`.
- `HostName`: The server's IP address.
- `User`: Your username for the SSH connection.

### 3. Connect Using Your Alias

Now you can connect to your server using the alias you configured:

```bash
ssh raspberry
```
