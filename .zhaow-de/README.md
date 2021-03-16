# Modifications Conducted

## Background

Since SoftEther 4->5 upgrade, MSCHAPv2 is enabled by default for the L2TP VPN. However, for the DataScience toolchains
we need to enforce the inner PPP protocol to PAP, because that is the only chance we get to let SoftEther transmit the
`User-Password` attribute to the backend Radius server for the upstream flow (VPN Client -> SoftEther -> Radius -> PAM -> Keycloak).

The modification is brutally disabling MSCHAPv2 to let SoftEther and FreeRADIUS negotiate a proper `Auth-Type`.

Moreover, the documentation of SoftEther 5 has a big room for improvement. The described procedures at the [README.md](../README.md)
and [BUILD_UNIX.md](../src/BUILD_UNIX.md) do not lead to a working binary for Windows or Linux.

## Procedures

### For Debian 10

```bash
apt install -y --no-install-recommends cmake cmake-data make gcc g++ libncurses5-dev libssl-dev libsodium-dev libreadline-dev zlib1g-dev pkg-config

git clone git@github.com:zhaow-de/SoftEtherVPN.git
cd SoftEtherVPN
git submodule update --init --recursive
```

Verify the modification is still in-place:
function `NewPPPSession` at `src/Cedar/Proto_PPP.c` should have:
```c
    p->EnableMSCHAPv2 = false;
```

Compile the software:
```bash
./configure
make clean -C build
make -C build
```

"Install" the software:
```bash
mkdir -p /opt/softether/{vpnserver,vpnbridge,vpnclient,vpncmd}
chmod +x build/hamcore.se2 build/libcedar.so build/libmayaqua.so build/vpnserver build/vpnbridge build/vpnclient build/vpncmd
cp build/hamcore.se2 build/libcedar.so build/libmayaqua.so build/vpnserver /opt/softether/vpnserver/
cp build/hamcore.se2 build/libcedar.so build/libmayaqua.so build/vpnbridge /opt/softether/vpnbridge/
cp build/hamcore.se2 build/libcedar.so build/libmayaqua.so build/vpnclient /opt/softether/vpnclient/
cp build/hamcore.se2 build/libcedar.so build/libmayaqua.so build/vpncmd /opt/softether/vpncmd/
```

Place a daily cronjob to rotate the logs
```shell
cp .zhaow-de/cron-softether-logs /etc/cron.daily/softether-logs
chmod +x /etc/cron.daily/softether-logs
```

For VPN server:
```shell
cp .zhaow-de/softether-vpnserver.service /etc/systemd/system/
systemctl enable softether-vpnserver
```

For VPN bridge:
```shell
cp .zhaow-de/softether-vpnbridge.service /etc/systemd/system/
systemctl enable softether-vpnbridge
```

For VPN client:
```shell
cp .zhaow-de/softether-vpnclient.service /etc/systemd/system/
systemctl enable softether-vpnclient
```
