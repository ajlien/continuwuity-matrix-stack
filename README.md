# Continuwuity Matrix Stack: a "batteries included" social space using Matrix

This is a quick-and-dirty playbook for deploying a self-hosted social space using Matrix.

The intended use case is a private, invite-only community resembling Discord or Slack. It is very performant for the small community sizes tested, but should continue to be relatively resource light for larger communities if federation with other Matrix homeservers remains disabled.

## Why use this?
Plenty of excellent guides for self-hosting Matrix already exist. These typically demonstrate deployments that have one or more of the following useful traits, but not _all_ of them:
- **Matrix homeserver and reverse proxy communicate over Unix socket instead of TCP.** Since they are on the same machine, this can yield a substantial performance improvement.
- **MatrixRTC included.** This provides out-of-the-box voice and video chat support through LiveKit.
- **Web client included.** Users can register and connect through a client that _you_ host, too - no need to trust a third-party app or web client.
- **Reverse proxy runs in its own Docker container.** A surprising number of walkthroughs assume that the _homeserver_ (and optionally LiveKit) run in Docker containers, but the reverse proxy server doesn't!
- **Federation disabled.** Counterintuitive for most Matrix self-host guides, but useful if the goal is to set up an invite-only, private social space e.g. for friends and family. There are some non-obvious configuration changes required, especially if using MatrixRTC.

## Services included
- **continuwuity**: a Matrix homeserver.
- **LiveKit**: enables voice and video calls, including voice rooms and screen streaming.
- **Cinny**: a web-based Matrix client with a UI resembling Slack or Discord.
- **Caddy**: a reverse proxy to properly route requests among the above and handle TLS certificates.

## Prerequisites
1. A host machine available, e.g. a VPS or homelab server, with a static public IP address.
2. Docker engine installed on the host machine.
3. Firewall for the host machine has open inbound ports:
    - 80
    - 443
    - 8448
    - 7880:7881
    - 50100:50200 (UDP)
    - 8081
    - 3478 (UDP)
    - 50300:50400 (UDP)
4. A domain name registered, e.g. `example.com`.
5. Public DNS records pointing to the registered domain:

| Type    | From                   | To                         |
|---------|------------------------|----------------------------|
| `A`     | `example.com`          | host's public IPv4 address |
| `AAAA`  | `example.com`          | host's public IPv6 address |
| `CNAME` | `matrix.example.com`   | `example.com`              |
| `CNAME` | `social.example.com`   | `example.com`              |
| `CNAME` | `livekit.example.com`  | `example.com`              |
6. Data backups configured as appropriate. _Do note that any end-to-end encrypted content may be unrecoverable if the entire docker-compose cluster is stopped and started again;_ more investigation is needed to determine how this happens.

## How to use
1. Clone this repository on the host machine. Navigate to this directory.
2. In all files, replace all references to `mydomain.com` with your domain e.g. `example.com`.
3. In `continuwuity.toml`, generate a unique token required for registration, and update the value in the line `registration_token = "your_servers_unique_registration_token"` with it.
4. Generate a unique key/value pair for LiveKit e.g. [using](https://continuwuity.org/calls/livekit.html#2-services) `docker run --rm livekit/livekit-server:latest generate-keys`. In `docker-compose.yml` and `livekit.yml`, replace `MyLiveKitKey` with the key and `MyLiveKitKeyValue` with the value.
5. Update `/etc/docker/daemon.json` by adding to the file
    ```
    {
        "userland-proxy": false
    }
    ```
    as this prevents runaway memory usage.
5. Run `sudo docker compose up -d`.
6. Wait for certificates to be generated.
7. Create the first user, which is automatically an admin user, by running `sudo docker exec -it continuwuity users create-user <username> <password>`, replacing `<username>` and `<password>` with your desired values. Note that if using a username _other than_ "admin", you should change the line in `static/well-known/matrix/support` to reflect the actual admin username.
8. Test the web client by connecting to `social.example.com` with a web browser and attempting to log in with the admin credentials from the previous step.
9. When inviting others to the server, share with them the _manually specified registration token_ from step 3 and have them register on `social.example.com` or with an alternative Matrix client of their choice.

## See also
- [linkpy's repo for c10-livekit-docker-compose](https://github.com/linkpy/c10y-livekit-docker-compose)
- [Official continuwuity documentation](https://continuwuity.org/)
- [Nex's guide for setting up continuwuity](https://nexy.blog/guides/Matrix/setting-up-continuwuity/0-intro/)
- [Tom Foster's guide for setting up continuwuity](https://tomfos.tr/matrix/continuwuity/)