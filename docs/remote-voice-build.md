# RustDesk 1.5.0 — Windows remote voice build

This custom Windows x64 build lets an authenticated remote operator click the
Windows RustDesk connection window, including the **Accept**, **Audio input**,
and **Stop voice call** controls. The controller can be the standard Mac app.

## What changes

The opt-in Cargo feature `remote-cm-control` sets the Windows default
`allow-remote-cm-modification=Y` through RustDesk's existing local settings layer.
An explicit `N` from user configuration or a policy override still takes precedence.
Without the feature, or on other platforms, the original behavior remains.

This option applies to the whole connection management window, including its
connection approval and permission buttons. It is not limited to the voice button.
The call still requires an explicit Accept click. The existing visible call state,
Stop button, connection authentication, and OS microphone permissions remain.

The application retains the RustDesk name and configuration location so existing
server settings and device configuration can be reused. This first build adds
remote voice controls; it does not yet add ClientX branding or a device directory.
No server address, password, or private key is embedded in this build.

## Install and try it

1. Download the Windows x64 build artifact and extract it. It includes a portable
   `.exe`, an `.msi` installer, build information, and SHA-256 checksums.
2. On Windows, close the existing RustDesk app before trying the custom executable.
   For unattended access, install the custom build and ensure its service is running.
   Replacing the running remote access service may disconnect your current session.
3. Connect from the Mac using the Windows machine's usual ID and authentication.
4. Start **Voice call** from the Mac session toolbar.
5. In the remote Windows desktop, bring the RustDesk connection window forward and
   click **Accept**. Use **Audio input** to select the Windows microphone if needed.
6. Verify speech in both directions, then click **Stop voice call** and confirm that
   the call ends. Confirm that Dismiss leaves the microphone inactive for that call.

Both operating systems must allow RustDesk to use the microphone. System sound
forwarding and the two-way voice call are separate functions.

The files are unsigned: no publisher signing certificate is configured for this
fork. They report RustDesk 1.5.0; `BUILD-INFO.txt` identifies the custom source commit.
An official RustDesk update will replace this custom behavior; rebuild this fork
when updating its RustDesk base.

## Rebuild

Run the **Windows remote voice client** GitHub Actions workflow on the
`codex/windows-voice-support` branch. It uses standard Ubuntu/Windows runners,
pins the upstream 1.5.0 toolchain and dependencies, builds a Windows x64 client,
runs the custom default/override regression test, and uploads an artifact.
It does not publish a release or invoke a signing service. For a packaging-only
retry after the test has passed with identical Rust sources, the manual workflow
allows turning off `run_permission_test` to avoid repeating that compilation.

For a Windows build environment set up like the workflow:

```powershell
cargo build --locked --release --lib --features flutter,hwcodec,vram,remote-cm-control
python build.py --portable --flutter --skip-cargo --skip-portable-pack --hwcodec --vram
```

The workflow additionally generates the Flutter bridge, packages the portable
executable and MSI, and records checksums. Build artifacts are retained for 30 days.
The repository and `LICENCE` provide the corresponding modified source under the
upstream GNU AGPL v3 license. Keep the license and corresponding source available
when distributing this custom build.

## Regression surface and validation

- `Cargo.toml`: one opt-in feature; existing default features are unchanged.
- `src/common.rs`: Windows custom-client initialization adds a local default only
  when that feature is enabled. This uses the existing connection-window input
  guard and preserves explicit disabling. A focused regression test checks both
  the feature's default and the explicit disable override.
- `.github/workflows/windows-remote-voice.yml`: new build pipeline derived from
  upstream's Windows Flutter build. Existing upstream workflows are unchanged.

No Flutter UI, audio transport, authentication, server, or submodule revision is
modified. Physical microphone/speaker round-trip testing requires the target Mac
and Windows computer; a successful CI build alone does not establish audio quality.
