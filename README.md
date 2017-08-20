The latest versions of Docker for Windows and Docker for Mac can build non-native architecture containers. I reached out to Anil Madhavapeddy, an engineer at Docker, asking about why it wasn't the case on Linux, turns out [Linux can too](https://twitter.com/avsm/status/885028269397090306), there's just an issue with some distributions of qemu-user-static that's disallowing the functionality. These files enable (likely among other things) the ability to build `arm32v7/*` Docker images on a Linux host of a different architecture without putting qemu-*-static binaries in the image itself. (Only tested on Debian 9)

### How do I know if I need them?

Run the following command:

`systemctl status systemd-binfmt`

If you see output like the following [note the `start condition failed` part]:

```
computermouth@debian:~$ systemctl status systemd-binfmt
● systemd-binfmt.service - Set Up Additional Binary Formats
   Loaded: loaded (/lib/systemd/system/systemd-binfmt.service; static; vendor preset: enabled)
   Active: inactive (dead) since Thu 2017-07-13 00:06:51 PDT; 5s ago
Condition: start condition failed at Thu 2017-07-13 00:06:51 PDT; 5s ago
           ├─ ConditionDirectoryNotEmpty=|/lib/binfmt.d was not met
           ├─ ConditionDirectoryNotEmpty=|/usr/lib/binfmt.d was not met
           ├─ ConditionDirectoryNotEmpty=|/usr/local/lib/binfmt.d was not met
           ├─ ConditionDirectoryNotEmpty=|/etc/binfmt.d was not met
           └─ ConditionDirectoryNotEmpty=|/run/binfmt.d was not met
     Docs: man:systemd-binfmt.service(8)
           man:binfmt.d(5)
           https://www.kernel.org/doc/Documentation/binfmt_misc.txt
  Process: 12402 ExecStart=/lib/systemd/systemd-binfmt (code=exited, status=0/SUCCESS)
 Main PID: 12402 (code=exited, status=0/SUCCESS)
      CPU: 0

```

...you will need the files here in one of the above directories. Personally, I've been using them in `/lib/binfmt.d`.

### How do I install?

```
git clone https://github.com/computermouth/qemu-static-conf.git
sudo mkdir -p /lib/binfmt.d
sudo cp qemu-static-conf/*.conf /lib/binfmt.d/
sudo systemctl restart systemd-binfmt.service
```

### How do I test if it's working

NOTE: You will need to `sudo docker` below if you haven't set up the appropriate [permissions](https://docs.docker.com/engine/installation/linux/linux-postinstall/).

```
echo -e "FROM arm32v7/debian:stretch-slim\n\nRUN echo 'it works'" > Dockerfile
docker build .
```

If the container succeeds to build without error, you're good to go!

### Why isn't this built into Debian/Ubuntu (but it is in Fedora)?

I'm not sure, that systemd service is distributed with them out of the box, but Fedora supplies these files with their qemu-user-static package, while Debian and Ubuntu don't. I've submitted a bug report here: https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=868217
