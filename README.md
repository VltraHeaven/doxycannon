# doxycannon

[![CodeFactor](https://www.codefactor.io/repository/github/audibleblink/doxycannon/badge)](https://www.codefactor.io/repository/github/audibleblink/doxycannon)

Doxycannon takes a pool of OpenVPN files and creates a Docker container for each one that binds a
socks server to a host port. Combined with proxychains, this creates your very own cheap and fast
private botnet.

## Prerequisites
- Install the required pip modules:
```sh
pip install -r requirements.txt
```
- Ensure docker is installed and enabled. Refer to the [Wiki](../../wiki/installing-docker)
for installation instructions on Kali/Debian
- `proxychains4` is required for interactive mode

## Setup
- Create an `auth.txt` file with your ovpn credentials in `VPN`. Format is:
  ```txt
  username
  password
  ```
- Fill the VPN folder with `*.ovpn` files and ensure that the `auth-user-pass` directive
  in your `./VPN/*.ovpn` files says `auth-user-pass auth.txt`
   - Watch [this wiki section](../../wiki#getting-started-with-vpn-providers) for installation tips
     on individual VPN providers
- Run `./doxycannon.py --build` to build your image with your OVPN files
  - `--build` will need to be run on code changes or when you modify the `VPN` folder's contents

## Usage

_note: the way proxychains seeds its PRNG to choose a random proxy is not fast enough to ensure
each subsequent request goes out through a different IP. You may get about between 2-10 requests
being made from the same IP. If this is unacceptable, check out my fork at
http://github.com/audibleblink/proxychains_

### One-off, random commands
While your containers are up, you can use proxychains to issue commands through random proxies

```sh
proxychains4 -q curl -s ipconfing.io/json
proxychains4 -q hydra -L users.txt -p Winter2018 manager.example.com -t 8 ssh
proxychains4 -q gobuster -w word.list -h http://manager.example.com
```

### GUI Tools

Use the `--single` flag to bring up your proxies and create a proxy rotator.

```sh
~/code/doxycannon  proxy_start                                                                                                             10m
❯❯ ./doxycannon.py --single
[+] Writing HAProxy configuration
[+] Writing Proxychains4 configuration
Starting: Mexico
Starting: Germany
Starting: Brazil
Starting: Japan
[+] All containers have been issued a start command
[*] Image doxyproxy built.
[*] Staring single-port mode. Ctrl-c to quit
^C
[*] doxyproxy was issued a stop command
[*] Your proxies are still running.
```

To see what's happening, checkout out the [haproxy](haproxy) folder. Essentially, one is building 
a layer 4 load-balancer between all the VPNs. This will allow you rotate through your proxies from
a single port which means you can point your browsers or BURPSuite instances at it and have every
request use a different VPN.

### Specific SOCKS proxies
If you want to use a specific proxy, give your utility the proper SOCKS port.

Example: To make a request through Japan, use `docker ps` and find the local port to which the
Japanese proxy is bound.

Configure your tool to use that port:

```sh
curl --socks5 localhost:50xx ipconfig.io/json
```

### Interactive
Once you've built your image and started your containers, run the utility with the `--interactive`
flag to get a bash session where all network traffic is redirected through proxychains4

```sh
./doxycannon.py --interactive
```

## Screenshots
![](https://i.imgur.com/jjHtk9L.png)
![](https://i.imgur.com/fLU4Mjx.png)

### Credit
[pry0cc](https://github.com/pry0cc/ProxyDock) for the idea

This was originally a fork of pry0cc's ProxyDock. It's been modified to an extent where less than
1% of the original code remains.

## TODO

- [X] Interactive mode
- [X] Python management script
  - [X] Faster Up/Down Container management
  - [ ] Allow for management of remote doxycannon installs through the Docker API

- [X] Dispatch server - (will allow GUI applications to use doxycannon)
  - [X] Creates a single local proxy server that dispatches through VPNs
