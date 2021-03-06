# binctr

Create fully static, including rootfs embedded, binaries that pop you directly
into a container. **Can be run by an unprivileged user.**

This is based off a crazy idea from [@crosbymichael](https://github.com/crosbymichael) who first embedded an image in a binary :D

### Building

```console
$ make static
Static container created at: ./bin/alpine
Run with ./bin/alpine

# building a different base image
$ make static IMAGE=busybox
Static container created at: ./bin/busybox
Run with ./bin/busybox
```

### Running

```console
$ ./alpine
$ ./busybox --read-only
```

### Running with custom commands & args

```console
# let's make an nginx binary
$ make static IMAGE=nginx
Static container created at: ./bin/nginx
Run with ./bin/nginx

$ ./bin/nginx nginx -g "daemon off;"

# But we have no networking! Don't worry we can fix this
# Let's install my super cool binary for setting up networking in a container
$ go get github.com/jfrazelle/netns

# now we can add this as a prestart hook
$ ./bin/nginx --hook prestart:netns nginx -g "daemon off;"

# let's get the ip file
$ cat .ip
172.19.0.10

Success!
```

### Usage

```console
$ ./bin/alpine -h
 _     _            _
| |__ (_)_ __   ___| |_ _ __
| '_ \| | '_ \ / __| __| '__|
| |_) | | | | | (__| |_| |
|_.__/|_|_| |_|\___|\__|_|

 Fully static, self-contained container including the rootfs
 that can be run by an unprivileged user.

 Embedded Image: alpine - sha256:70c557e50ed630deed07cbb0dc4d28aa0f2a485cf7af124cc48f06bce83f784b
 Version: 0.1.0
 GitCommit: 13fcd27-dirty

  -D	run in debug mode
  -console string
    	the pty slave path for use with the container
  -d	detach from the container's process
  -hook value
    	Hooks to prefill into spec file. (ex. --hook prestart:netns) (default [])
  -id string
    	container ID (default "nginx")
  -pid-file string
    	specify the file to write the process id to
  -read-only
    	make container filesystem readonly
  -root string
    	root directory of container state, should be tmpfs (default "/run/binctr")
  -t	allocate a tty for the container (default true)
  -v	print version and exit (shorthand)
  -version
    	print version and exit
```

## Caveats

**Caps the binary needs TO UNPACK AND SET THE RIGHT PERMS ON THE ROOTFS FOR THE USERNS USER**

- **CAP_CHOWN**: chown the rootfs to the userns user
- **CAP_FOWNER**: chmod rootfs
- **CAP_DAC_OVERRIDE**: symlinks

**These can be dropped after the rootfs is unpacked and chowned.**

-------

**Caps for libcontainer**

- **CAP_SETUID**, **CAP_SETGID**: so we can write to `uid_map`, `gid_map`, in
  `nsexec.c`
See: http://man7.org/linux/man-pages/man7/user_namespaces.7.html
