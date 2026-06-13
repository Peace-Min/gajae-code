# GajaeCode Air-Gapped Windows Installer

This project builds a single Windows installer executable for the internal
Anthropic-compatible gateway.

Deployment settings are supplied only at build time and embedded in the final
installer. Internal gateway addresses are not committed to the public source.
The API key environment variable is `GJC_INTERNAL_API_KEY`.

The API key is requested at install time. It is stored as a Windows user
environment variable and is not written to `models.yml` or committed to Git.

Git Bash is an existing required dependency. The installer detects it and does
not install or update Git for Windows.

## Build on the connected staging PC

Obtain and verify the official `gjc-windows-x64.exe`, then run:

```powershell
.\deploy\airgap-windows-installer\build.ps1 `
  -GjcBinary C:\staging\gjc-windows-x64.exe `
  -GjcLinuxBinary C:\staging\gjc-linux-x64 `
  -WslRootfs C:\staging\gajaecode-wsl-rootfs.tar `
  -WslKernelMsi C:\staging\wsl_update_x64.msi `
  -GatewayBaseUrl http://internal-gateway:8080/anthropic `
  -ModelId internal-model-id
```

Outputs:

```text
deploy\airgap-windows-installer\artifacts\GajaeCode-Airgap-Setup.exe
deploy\airgap-windows-installer\artifacts\SHA256SUMS.txt
```

Commit the final installer and checksum to the internal deployment repository
or attach them to an internal Git release. Do not commit an API key.

## Run on the air-gapped PC

1. Run `GajaeCode-Airgap-Setup.exe`.
2. Enter the server-issued bearer API key.
3. Press Enter or click **설치 및 설정**.

The installer:

1. verifies and installs the embedded GJC binary;
2. registers the API key as a user environment variable;
3. adds GJC to the user `PATH`;
4. writes the internal provider and all four agent role mappings;
5. disables update checks, external web search, and browser tools;
6. installs the bundled default workflow skills;
7. tests the gateway and a real GJC model request.
8. enables WSL2 and resumes after reboot when required;
9. imports a tmux-ready offline Linux root filesystem;
10. installs Linux GJC and creates a `GajaeCode tmux` desktop launcher.
11. creates a disposable verification project and runs a real agent edit/test
    workflow;
12. writes a sanitized JSON evidence file and self-contained HTML report.

Verification outputs:

```text
%USERPROFILE%\GajaeCode-Verification
%USERPROFILE%\.gjc\verification\latest.json
%USERPROFILE%\Desktop\GajaeCode-Installation-Report.html
```

Existing `models.yml` and `config.yml` files are timestamp-backed up before
the managed configuration is written.

## Failure diagnostics

Every attempt creates sanitized recovery artifacts:

```text
%LOCALAPPDATA%\GajaeCode\diagnostics\install-<timestamp>.log
%LOCALAPPDATA%\GajaeCode\diagnostics\latest-install-state.json
%LOCALAPPDATA%\GajaeCode\diagnostics\RECOVERY.md
```

The state file records the last installation stage and error so another agent
session can continue diagnosis. API key values are excluded and redacted.
