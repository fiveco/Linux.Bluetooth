# Publishing Linux.Bluetooth to the internal NuGet server

This is a Fiveco fork of [SuessLabs/Linux.Bluetooth](https://github.com/SuessLabs/Linux.Bluetooth).
NuGet packages are built and published manually. The GitHub Actions workflow only runs unit tests.

## Versioning

- `Directory.Build.props` — upstream file, defines `ReleaseVersion` (e.g. `6.0.0-pre4`). **Do not edit.**
- `Directory.Build.targets` — FCO-only file, defines `FCOVersionSuffix`. Edit this for each release.

The final package version is `$(ReleaseVersion)$(FCOVersionSuffix)`:

| Scenario | `FCOVersionSuffix` | Package version |
|---|---|---|
| First release on upstream version | `-fco` | `6.0.0-pre4-fco` |
| Subsequent iteration | `-fco-1`, `-fco-2`, … | `6.0.0-pre4-fco-1`, … |

## Prerequisites

- .NET SDK installed locally
- Access to the internal Gitea NuGet server
- Your Gitea personal access token (needs `package:write` scope)

## Steps

**1. Set the version**

The upstream version comes from `ReleaseVersion` in `Directory.Build.props` — do not edit that file.
The FCO suffix is defined in `Directory.Build.targets` (FCO-only file, not present upstream):

```xml
<FCOVersionSuffix>-fco</FCOVersionSuffix>
```

Change it to `-fco-1`, `-fco-2`, etc. for iterations. The final package version is `$(ReleaseVersion)$(FCOVersionSuffix)`.

**2. Build and pack**

*Visual Studio:* right-click the `Linux.Bluetooth` project in Solution Explorer → **Pack**. The `.nupkg` is written to `output/`.

*CLI:*
```bash
dotnet pack src/Linux.Bluetooth/Linux.Bluetooth.csproj \
  --configuration Release \
  --output ./nupkg
```

**3. Publish**

*CLI only* — Visual Studio has no built-in push for custom NuGet sources.

```bash
dotnet nuget push ./nupkg/Linux.Bluetooth.<version>.nupkg \
  --source "https://<gitea-host>/api/packages/<org>/nuget/index.json" \
  --api-key "<your-token>"
```

If the `gitea` source is already configured in your local NuGet config:

```bash
dotnet nuget push ./nupkg/Linux.Bluetooth.<version>.nupkg \
  --source "gitea" \
  --api-key "<your-token>"
```
