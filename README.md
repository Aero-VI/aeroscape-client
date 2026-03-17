# AeroScape

An Old School RuneScape private server built on a RuneLite fork with a custom C# game server.

AeroScape is a from-scratch RSPS (RuneScape Private Server) that connects a modified RuneLite client to a custom-built game server. The project implements the full OSRS network protocol — JS5 cache serving, login authentication with RSA/ISAAC/XTEA cryptography, and game world communication.

## What Works

- **Full game cache delivery** — Custom JS5 cache server written in C# serves real OSRS rev 236 cache data. Sector-chain disk reading, master checksum table, CRC32 validation, 0xFF block framing — all implemented from protocol documentation and reverse engineering.

- **Login screen** — The OSRS login screen renders fully. Players can enter credentials and click Login.

- **Login authentication** — Complete OSRS rev 236 login protocol: RSA-encrypted credential block with ISAAC cipher seed exchange, XTEA-encrypted metadata, OTP authentication fields, and the full 36-byte success response format.

- **Runtime RSA key override** — The gamepack's RSA public keys are replaced at runtime using `sun.misc.Unsafe` reflection, allowing the server to decrypt login blocks with its own 1024-bit key pair. Handles multiple RSA key locations across obfuscated gamepack classes.

- **Host validation bypass** — Binary-patched domain validation in the gamepack to accept our server domain instead of runescape.com. Patches tracked across obfuscation changes between revisions.

- **Dual-port networking** — Server listens on both port 43594 (standard OSRS) and 443 (for JS5 connections that use the HTTPS port for firewall traversal).

## Architecture

```
Client (RuneLite fork, Java)          Server (C# .NET 8)
┌─────────────────────────┐           ┌──────────────────────┐
│ Modified RuneLite        │           │ JS5 Cache Handler    │
│ - RSA key override       │    TCP    │ - Rev 236 cache      │
│ - Host validation bypass │◄────────►│ - Sector-chain reads │
│ - AeroScape config       │  :43594  │ - Master index       │
│ - Embedded jav_config    │   :443   │                      │
│                          │           │ Login Handler        │
│ Gamepack (rev 236)       │           │ - RSA decryption     │
│ - Injected client        │           │ - ISAAC init         │
│ - Patched classes        │           │ - XTEA decryption    │
│                          │           │ - Rev 236 response   │
│ RuneLite plugins         │           │                      │
│ - GPU rendering          │           │ Game World (planned) │
│ - All standard plugins   │           │                      │
└─────────────────────────┘           └──────────────────────┘
```

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Client | Java 17+, RuneLite, Gradle |
| Server | C# / .NET 8 |
| Cache | OSRS rev 236 from OpenRS2 archive |
| Cryptography | RSA 1024-bit, ISAAC cipher, XTEA |
| DNS | Cloudflare (play.aeroverra.com) |
| Hosting | Proxmox VE, Debian 12 |

## Building

### Client
```bash
./gradlew :client:shadowJar
# Post-build: apply host validation patches
python3 scripts/patch_hosts.py
```

### Server
```bash
cd AeroScape.LoginServer
dotnet build
dotnet run
```

See [docs/RSPS-BUILD-GUIDE.md](docs/RSPS-BUILD-GUIDE.md) for the complete build guide including reverse engineering notes, protocol documentation, and a revision update checklist.

## Roadmap

- [x] JS5 cache server
- [x] Login screen rendering
- [x] Login authentication (RSA + ISAAC + XTEA)
- [x] Host validation bypass
- [x] Runtime RSA key override
- [ ] Game world server (player spawning, map rendering)
- [ ] Player movement and interaction
- [ ] NPC spawning
- [ ] Chat system
- [ ] Skills and combat

## Project History

Built in a single 16-hour session. 26 versions shipped. From "client crashes on launch" to "fully authenticated login" in one day.

## Related

- **Server:** [Aero-VI/aeroscape-server](https://github.com/Aero-VI/aeroscape-server)
- **Documentation:** [docs/RSPS-BUILD-GUIDE.md](docs/RSPS-BUILD-GUIDE.md)

## License

This project is for educational and private server purposes. RuneLite is licensed under BSD 2-Clause.
