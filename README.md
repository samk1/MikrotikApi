# MikrotikApi

A .NET library for automating the configuration of [MikroTik](https://mikrotik.com/) routers via their built-in API.

## What is this?

This is a personal home-automation project I built to scratch a real itch: I own a MikroTik router (RouterOS) and wanted a repeatable, code-driven way to apply configuration changes instead of clicking through the GUI every time.

MikroTik routers expose a binary TCP API on port 8728. This library wraps that protocol in a clean C# interface so you can read and write router configuration from any .NET application — think of it as a low-level SDK for RouterOS.

## Features

- **TCP connection & login** — handles the RouterOS two-step MD5-challenge authentication handshake.
- **Command execution** — send any RouterOS API command and receive structured results.
- **Fluent `CommandBuilder`** — build commands with attributes without worrying about wire-format details.
- **Expressive `QueryBuilder`** — filter results with typed comparisons (`Equals`, `LessThan`, `GreaterThan`, `Exists`, `NotExists`) and compose them with boolean operators (`!`, `|`, `&`).
- **Strongly-typed responses** — results come back as a `ResponseData` list of `ResponseItem` dictionaries, ready to bind to your own models.

## Quick start

```csharp
using MikrotikApi;

// Connect and authenticate
using var client = new Client("192.168.88.1");
client.Login("admin", "password");

// Simple command – list all IP addresses
var addresses = client.DoCommand("/ip/address/print");
foreach (var item in addresses)
    Console.WriteLine(item["=address"]);

// Command with attributes
client.DoCommand("/ip/firewall/address-list/add", new Dictionary<string, string>
{
    { "list",    "blocked" },
    { "address", "10.0.0.0/8" }
});

// Filtered query with CommandBuilder / QueryBuilder
var builder = new CommandBuilder("/ip/firewall/address-list/print");
builder.Query(q => q.Equals("list", "blocked"));
var blocked = client.DoCommand(builder);
```

## Why this is relevant to DevOps

| Concept | How it shows up here |
|---|---|
| Infrastructure as Code | Router config applied programmatically, not by hand |
| Network automation | RouterOS API protocol implementation from scratch |
| Idiomatic C# / .NET | `IDisposable`, generics, operator overloading, fluent builders |
| Protocol engineering | Binary framing (length-prefix encoding), MD5 challenge auth |
| Testability | Unit tests for the query DSL and the wire-format encoder |

## Project structure

```
MikrotikApi/
├── Client.cs           # TCP connection, auth, send/receive
├── CommandBuilder.cs   # Fluent command construction
├── QueryBuilder.cs     # Boolean-composable query filters
├── QueryOperations.cs  # Query operation helpers
├── Protocol/           # Wire-format words, sentences, responses
└── Tests/              # Unit tests
```

## Requirements

- .NET Framework 4.5.2 or later (the project targets 4.5.2; porting to .NET Core / .NET 5+ requires replacing `MD5CryptoServiceProvider` with `MD5.Create()`)
- A MikroTik router with the API service enabled (`/ip service enable api`)

## Disclaimer

This is a toy project built for fun and learning. It is not production-hardened — use it at your own risk on home lab kit.
