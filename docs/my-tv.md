# This fork: Samsung UA55TU8300 (2020)

This fork adapts CleanFrame for a **Samsung UA55TU8300UXKE**, a 2020 55-inch curved Crystal UHD TV. It is not a Frame TV. Its software version, T-NKLUABC-2743.0, belongs to Samsung's 2020 platform, which runs **Tizen 5.5**.

## What changed from upstream

| Change | Why |
| --- | --- |
| `app/config.xml` declares `required_version="5.5"` | Upstream declares Tizen 6.5, which a Tizen 5.5 TV would refuse to install. |
| Art Mode control removed from Home | Only Frame TVs have Art Mode (`org.tizen.art-app`). Home now shows **All apps** and **Settings**. |
| Glass field uses a 2D `translate`, and `.ut-lens` has an opaque `-webkit-mask` | Older Tizen browser engines ignore the rounded clip on a 3D-transformed layer, so the glass spilled past the rounded corners. Taken from upstream PR #11, tested on a 2019 Frame running Tizen 5.0. |

Upstream already replaced flex `gap` and `inset`, which older engines don't support. The runtime JavaScript is ES5. Browser checks pass, but nobody has tested this build on the TV yet.

**Not attempted:** native Home/startup replacement (`scripts/native-home/`). Those helpers are guarded to the maintainer's 2024 Frame firmware and are unsafe on this TV. Open CleanFrame from the TV's Apps list instead.

## Installing it on the TV

You need a Mac or Linux computer on the same Wi-Fi/network as the TV. The signing scripts don't run on Windows.

### 1. Prepare the computer

1. Install [Tizen Studio](https://developer.samsung.com/smarttv/develop/getting-started/setting-up-sdk/installing-tv-sdk.html). In its Package Manager, also install the **Samsung TV Extension** and the **Samsung Certificate Extension**.
2. Install Node.js 22 (LTS), Python 3.10+ and Git.
3. Get this fork and its tools:

   ```sh
   git clone https://github.com/mwakatumbula/CleanFrame.git
   cd CleanFrame
   npm ci
   ```

### 2. Turn on Developer Mode on the TV

1. Find your computer's local IP address (for example `192.168.1.20`).
2. On the TV, open **Apps**. Using the remote's number keys, or the on-screen number pad, enter **1 2 3 4 5**.
3. Set **Developer Mode → On**, enter the computer's IP as **Host PC IP**, then press **OK**.
4. Restart the TV fully: hold the remote's power button until the TV switches off and on again.
5. Note the TV's IP address from **Settings → General → Network → Network Status → IP Settings**.

### 3. Connect and create certificates

1. In Tizen Studio, open **Device Manager**, add the TV's IP and connect. The command-line equivalent is `sdb connect <TV_IP>`.
2. Open **Certificate Manager**, create a new **Samsung → TV** certificate profile and sign in with your Samsung account. When it asks for the device ID (DUID), pick the connected TV from the list. Remember the profile name.
3. In Tizen Studio, create an empty **TV → Web Application** project. Its `config.xml` gives you an SDK-generated **package ID**, a 10-character code like `Ab12Cd34Ef`. You only need that ID; you can delete the empty project afterwards.

### 4. Tell CleanFrame about your setup

Create two private files. Git ignores the `.local/` folder.

`.local/signing.json`, based on `signing.example.json`:

```json
{
  "sdk_path": "/Users/you/tizen-studio",
  "profile": "YOUR_CERTIFICATE_PROFILE",
  "package_id": "Ab12Cd34Ef",
  "app_id": "Ab12Cd34Ef.CleanFrame",
  "password_file": null
}
```

`.local/device.json`:

```json
{"host": "YOUR_TV_IP"}
```

### 5. Build, install and open

```sh
npm run package      # builds and signs dist/CleanFrame.wgt with your certificate
npm run tv:install   # installs it on the TV
npm run tv:launch    # opens it
```

CleanFrame then appears in the TV's **Apps** list, so you can open it from there at any time.

## First things to check on the TV

- Use the arrow keys, **OK** and **Back** to move around Home. Press **Up** to reach All apps and Settings.
- Some default apps (Stremio, ABC iview, SBS On Demand) are Australian and probably aren't installed on this TV. Open **Settings → Apps on Home**, or go to **All apps** and press Up to reach **Apps on Home**. From there you can remove them and add the apps you actually use.
- Check that the rounded glass highlight stays inside its corners and that the fonts look right.
- Inside CleanFrame's Settings, the **Samsung Home** and **TV settings** shortcuts use app IDs taken from a 2024 TV. If either fails on this TV, use the remote's own Home and Settings buttons instead.

If something looks wrong, take a photo of the screen and include it when reporting the problem.
