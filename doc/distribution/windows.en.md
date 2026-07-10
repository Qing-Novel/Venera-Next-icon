# Windows Distribution

Chinese version: [windows.zh.md](windows.zh.md)

## Release Artifacts

Windows stable releases are built by `.github/workflows/main.yml` through the `完整构建` workflow:

- `VeneraNext-<version>-windows-installer.exe`: Inno Setup installer, suitable for winget.
- `VeneraNext-<version>-windows.zip`: portable package, suitable for manual download and extraction.

winget uses the installer by default, not the portable package. After the package is accepted by winget, users can install or upgrade with:

```powershell
winget install CyrilPeng.VeneraNext
winget upgrade CyrilPeng.VeneraNext
```

## Generate Winget Manifest

When a stable release tag is published, the main release workflow generates the `winget_manifest` artifact. You can also manually run the `准备 Winget Manifest` workflow and input an existing stable tag, for example `v1.10.2`.

The manual workflow only generates the manifest artifact by default. To also create a PR to `microsoft/winget-pkgs`:

1. Configure `WINGET_PKGS_TOKEN` in repository secrets. The token needs enough permission to fork the repository and create a PR to `microsoft/winget-pkgs`.
2. Enable `submit_pr` when running the `准备 Winget Manifest` workflow.

The workflow uses `.github/scripts/submit_winget_manifest_pr.py` to update the manifest branch in the `CyrilPeng/winget-pkgs` fork through the GitHub API and create a PR, avoiding a full clone of the large `winget-pkgs` repository.

Local generation command:

```powershell
python .github\scripts\generate_winget_manifest.py `
  --version 1.10.2 `
  --installer build\windows\VeneraNext-1.10.2-windows-installer.exe `
  --output build\winget `
  --print-path
```

The generated directory follows the winget-pkgs layout:

```text
build/winget/manifests/c/CyrilPeng/VeneraNext/<version>/
```

## Submit To winget-pkgs

For the first winget submission, submit the generated directory to the matching path in `microsoft/winget-pkgs` and create a PR. If `WINGET_PKGS_TOKEN` is configured, the `submit_pr` option of the `准备 Winget Manifest` workflow can do this directly.

After the package is already accepted, WingetCreate can be used for updates:

```powershell
wingetcreate update CyrilPeng.VeneraNext `
  -u https://github.com/CyrilPeng/venera-next/releases/download/v1.10.2/VeneraNext-1.10.2-windows-installer.exe `
  -v 1.10.2 `
  -t <GitHub PAT> `
  --submit
```

Do not submit winget manifests for `-rc` prerelease versions. winget should follow stable releases only.

## Notes

- Do not casually change the `AppId` in `windows/build.iss`; it affects how winget identifies installed apps.
- The installer filename must remain `VeneraNext-<version>-windows-installer.exe`; the manifest script validates this naming.
- If Windows ARM64 stable releases are added later, the winget installer manifest needs an `arm64` installer entry.
- Code signing is not currently required by the scripts, but should be prioritized for winget distribution to reduce SmartScreen and installation trust issues.
