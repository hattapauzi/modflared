# Modflared 1.12.2 API Diff Notes

## Direct-Connect Hook

- Modern source: `net.minecraft.client.gui.screens.ConnectScreen$1#run`.
- 1.12.2 target to verify in source: `net.minecraft.client.multiplayer.GuiConnecting$1#run`.
- Verified target: `net.minecraft.client.multiplayer.GuiConnecting$1#run`.
- Verified network factory: `NetworkManager.createNetworkManagerAndConnect(InetAddress, int, boolean)`.
- Expected network call to intercept: creation or opening of `net.minecraft.network.NetworkManager` for the remote address.
- Required behavior: compute tunnel status before opening the channel and substitute `127.0.0.1:<deterministic local port>` only when status is `USE`.
- Mixin: `dev.httxrafa.modflared.mixin.client.GuiConnectingThreadMixin`.
- Reroute rule: replace the original host/port with `RunningTunnel.Access.localWithRandomPort(route).getTunnelAddress()` only when tunnel status is `USE`.

## Server-List Ping Hook

- Modern source: `net.minecraft.client.multiplayer.ServerStatusPinger#pingServer`.
- 1.12.2 target to verify in source: `net.minecraft.client.network.ServerPinger#ping(ServerData)`.
- Verified target: `net.minecraft.client.network.ServerPinger#ping(ServerData)`.
- Verified network factory: `NetworkManager.createNetworkManagerAndConnect(InetAddress, int, boolean)`.
- Required behavior: route saved-server ping traffic through local cloudflared when TXT or forced-tunnel rules require it.
- Mixin: `dev.httxrafa.modflared.mixin.client.ServerPingerMixin`.
- Cosmetic parity: server-list row presentation is best-effort and not required for the first working path.

## Tunnel Lifecycle Hook

- Modern source: `net.minecraft.network.Connection#disconnect`.
- 1.12.2 target to verify in source: `net.minecraft.network.NetworkManager#closeChannel` and/or channel inactive handling.
- Verified target: `net.minecraft.network.NetworkManager#closeChannel`.
- Required behavior: close the associated `RunningTunnel` exactly once when the Minecraft connection closes.

## Forced Tunnel Config Load Path

- Modern path: `<gameDir>/modflared/forced_tunnels.json`.
- 1.12.2 target path: `<Minecraft.getMinecraft().mcDataDir>/modflared/forced_tunnels.json`.
- Required behavior: preserve JSON array of server address strings.
