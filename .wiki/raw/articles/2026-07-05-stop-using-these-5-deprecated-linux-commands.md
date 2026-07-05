---
title: "Stop using these 5 deprecated Linux commands (and what to use instead)"
source: "https://www.howtogeek.com/stop-using-these-deprecated-linux-commands-and-what-to-use-instead/"
type: articles
ingested: 2026-07-05
tags: [linux, command-line-tools, networking, iproute2, deprecated-tools]
summary: "How-To Geek article explaining five older Linux commands and their modern replacements: arp to ip neigh, ifconfig to ip address, iptables to nftables, which to shell builtins, and netstat to ss."
---

# [Stop using these 5 deprecated Linux commands (and what to use instead)](https://www.howtogeek.com/stop-using-these-deprecated-linux-commands-and-what-to-use-instead/)

Linux has hundreds of available commands, and unless you're keeping up with the latest news, changes to the default tools may slip under your radar. While it's true that Linux is very stable, and tool deprecations are rare, some of our much-beloved and well-known utilities changed years ago, and nobody informed us. I have five deprecated Linux commands and their replacements.

## Ditch arp for ip n

### A modern and colorful alternative

For those unaware, network hardware communicates with [MAC addresses](https://www.howtogeek.com/764868/what-is-a-mac-address-and-how-does-it-work/), but software uses [IP addresses](https://www.howtogeek.com/341307/how-do-ip-addresses-work/). When software makes a request, our computers ask the [local area network](https://www.howtogeek.com/353283/what-is-a-local-area-network-lan/) (LAN) which MAC address belongs to a particular IP address. This process is called an [ARP](https://www.howtogeek.com/813741/linux-arping-command/#the-arp-protocol) (Address Resolution Protocol) request.

A MAC address is a (typically) unique, 6-byte hexadecimal number that most LAN devices have burned into them-unless changed by the OS.

For as long as I've been on Linux, and even Windows, the command to interact with ARP was always "arp." But for around a decade now, the new norm has been to use `ip neigh` (neighbor) or `ip n` for short.

The "ip" command is part of the newer [iproute2](https://wiki.linuxfoundation.org/networking/iproute2) package, which superseded the aging [net-tools](https://wiki.linuxfoundation.org/networking/net-tools). The last release of net-tools was in 2021, and [talks about deprecation occurred as far back as 2009](https://lists.debian.org/debian-devel/2009/03/msg00780.html).

So if you're still using "arp," it's probably time to switch.

## Drop ifconfig for ip a

### ifconfig is legacy and development stopped long ago

The `ifconfig` command was likely one of the first you learned on Linux. It was easy to remember, and even easier to get useful information from. However, `ifconfig` is part of the now-deprecated net-tools package, and so development stopped for it long ago, leaving it without modern features and showing signs of age.

Its replacement ("ip a" or `ip address`), again, is part of iproute2, and it has a much sleeker, more colorful display:

![A terminal window displays the output of the ip a command, listing network interfaces with their addresses and states.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2026/02/a-terminal-window-displays-the-output-of-the-ip-a-command-listing-network-interfaces-with-their-addresses-and-states.png?q=70&fit=crop&w=825&dpr=1)

It's not just for viewing information either, because you can also use it to configure interfaces on the fly. These changes are volatile and don't last, so beware.

![A terminal window displays the help output of the ip address command, showing its usage and available options.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2026/02/a-terminal-window-displays-the-help-output-of-the-ip-address-command-showing-its-usage-and-available-options.png?q=70&fit=crop&w=825&dpr=1)

## Switch from iptables to nft

### It's more efficient and is the new norm

If you've configured a firewall on Linux, you've undoubtedly heard of [iptables](https://en.wikipedia.org/wiki/Iptables). It's a long-standing command that allows administrators to build up complex policy rules-e.g., filtering, mangling, and routing packets. It's one of those commands that I found hard to move on from because of the complexity of the role it fills. However, [Debian 10 made nftables the default](https://wiki.debian.org/nftables#:~:text=nftables%20is%20the%20default%20framework%20in%20use%20in%20Debian%20%28since%20Debian%2010%20Buster%29) in 2019, and [RHEL 9 formally deprecated iptables](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/9.0_release_notes/deprecated_functionality#deprecated-functionality_networking) in 2022- `iptables` is now consigned to the aging firmware on your talking toaster.

However, the Linux community isn't rash; there was clearly a replacement, and [nftables](https://en.wikipedia.org/wiki/Nftables) found its way into the kernel in 2014. It was a natural progression from its predecessor, with improvements in efficiency and reduced code complexity. If you don't write firewall policy rules, this will probably bore you, but if you do, you have no choice but to use it.

## Stop using which

### Shell builtins are less wasteful and recommended

Everybody has used this command at some point, and I still use it. In a nutshell, if you're looking for a binary path on your system, executing `which` tells you where it is. Although it isn't yet explicitly deprecated, other commands have superseded it.

One of its replacements is `type`, a shell built-in for [Bash](https://www.gnu.org/software/bash/manual/html_node/Bash-Builtins.html#index-type), [Zsh](https://linux.die.net/man/1/zshbuiltins#:~:text=type,v%2E), and possibly others:

![A terminal window shows two commands. The first runs type -f man, the second runs type -p man. Both return the path to man.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2026/02/a-terminal-window-shows-two-commands-the-first-runs-type-f-man-the-second-runs-type-p-man-both-return-the-path-to-man.png?q=70&fit=crop&w=825&dpr=1)

"type -p" behaves differently on Zsh. Check the Zsh manual.

Another alternative is to use `command -v`, which is also a built-in:

![A terminal window shows two commands. The first runs type -t command, returning builtin. The second runs command -v man, returning its path.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2026/02/a-terminal-window-shows-two-commands-the-first-runs-type-t-command-returning-builtin-the-second-runs-command-v-man-returning-its-path.png?q=70&fit=crop&w=825&dpr=1)

Since they're both built into the shell, they're much faster to execute from scripts because the shell doesn't need to fork the process.

## Switch from netstat to ss

### It's more powerful and easier to use

The last one on the list replaces a classic. [Netstat](https://www.howtogeek.com/513003/how-to-use-netstat-on-linux/) is also available on Windows, and it's another command I've used for decades. On Linux, `netstat -altpn` is long burned into my brainstem, but its newer replacement retains many of the commonly used flags:

![A Linux terminal shows the output of the ss ALTPN command. It's almost identical to its netstat counterpart.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2026/02/a-linux-terminal-shows-the-output-of-the-ss-altpn-command-it-s-almost-identical-to-its-netstat-counterpart.png?q=70&fit=crop&w=825&dpr=1)

It's an improvement in terms of speed and utility. The `ss` command pulls information directly from kernel interfaces, making it much snappier. It's not limited to network sockets either, because it can display information on Unix domain sockets:

![A terminal window displays the output of the ss -x command, listing Unix domain sockets and their connection details.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2026/02/a-terminal-window-displays-the-output-of-the-ss-x-command-listing-unix-domain-sockets-and-their-connection-details.png?q=70&fit=crop&w=825&dpr=1)

The "ss" command can do a great deal more. For example:

- `ss -t state listening`: View listening TCP sockets.
- `ss state syn-sent`: View connection status at various stages.
- `ss dport = :443`: View connections bound to a remote port.

The "ss" command is also part of the iproute2 package, and it's a clear upgrade on `netstat`.

---

A large chunk of the commands covered today are part of the iproute2 package, and so you should dig a little deeper into the utilities it provides because, ultimately, it's the replacement for many older, deprecated tools.

While most of the commands on Linux will remain the same for years, occasionally some change. It's always nice to have a refresher and introduce yourself to new tools. The older commands are far from perfect, so even small upgrades can be plenty useful.
