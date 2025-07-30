✅ 1. Remove old Docker versions >> to avoid package conflicts.
      apt module: Manages APT packages.
      state: absent: Ensures these packages are removed.
      loop: Applies the task to each item in the list:
      docker, docker-engine, docker.io: Older or alternative Docker versions.
      containerd, runc: Docker-related container runtimes.
  
✅ 2. Update apt cache >> Refresh the list of available packages from repositories.

✅ 3. Install required dependencies >> tools needed for Docker installation
      ca-certificates: Validates HTTPS connections.
      curl: Downloads files from URLs.
      gnupg: Manages cryptographic keys (for verifying package signatures).
      lsb-release: Detects Ubuntu version for conditiona

✅ 4. Create the keyrings directory >>  Ensure the GPG key is stored in the right directory
      Ubuntu now recommends placing trusted GPG keys in /etc/apt/keyrings

✅ 5. Download Docker’s GPG key >> 
      Download and convert Docker's GPG key into a format APT understands (.gpg or .asc)
      creates: Ensures this runs only once, unless the file is deleted

✅ 6. Set GPG key file permissions >> GPG key readable by all users (needed by APT)
      0644: Read/write for owner, read-only for group and others.

✅ 7. Add Docker repository >> Add Docker’s official APT repo to install Docker CE
      For Ubuntu 24.04 (Noble): Docker hasn't officially released a repo for Noble yet,
      so we use jammy (22.04), which is compatible.

✅ 8. Update apt cache again

✅ 9. Install Docker 
      docker-ce: Docker Community Edition daemon.
      docker-ce-cli: Command-line interface.
      containerd.io: Core container runtime.
      docker-buildx-plugin: Build tool plugin.

✅ 10. Install Docker compose