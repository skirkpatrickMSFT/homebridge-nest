<p align="center">
<img src="https://github.com/homebridge/branding/raw/latest/logos/homebridge-wordmark-logo-vertical.png" width="150">
</p>

<span align="center">

# homebridge-nest

**Nest Thermostat, Protect and Lock plug-in for [Homebridge](https://github.com/homebridge/homebridge)**

</span>

> [!IMPORTANT]
>
> **Homebridge v2.0 Compatibility**
>
> This is an unofficial fork of [chrisjshull/homebridge-nest](https://github.com/chrisjshull/homebridge-nest) (v4.6.10) updated to work with **Homebridge v2** (HAP-NodeJS v1).
>
> - `package.json → engines.homebridge` is set to `">=0.2.5 || ^2.0.0-beta.0"`
>
> **What was changed from upstream:**
> - Replaced deprecated `.on('get')` / `.on('set')` callback-style characteristic handlers with the v2-required `.onGet()` / `.onSet()` promise-based API
> - Updated `updateData()` to push fresh device values to HomeKit via `updateValue()` instead of the removed `.value` no-op trigger
> - Fixed the Nest-unreachable error-state handlers to use the HAP-NodeJS v1 API

---

Integrate your Nest Thermostat, Temperature Sensors, Nest Protect, and Nest x Yale Lock devices into your HomeKit system via Homebridge. Both Nest Accounts (pre-August 2019) and Google Accounts are supported.

Supports all Nest Thermostat, Protect, and Nest x Yale Lock models, including the EU/UK model of the Thermostat E with Heat Link and the October 2020 Nest Thermostat with mirror display.

> Cameras are not supported — see [homebridge-nest-cam](https://github.com/Brandawg93/homebridge-nest-cam) for that.

---

### Install the Plugin

> **Requirements:** Node.js v18 or later, Homebridge v1.6.0 or later (including v2).

Clone this repository and link it into your global Homebridge installation:

```shell
git clone https://github.com/skirkpatrickMSFT/homebridge-nest.git
cd homebridge-nest
npm install
npm link
```

Then restart Homebridge. On first run, check your logs — each device's `structureId` and `deviceId` are printed there.

> **Homebridge v1 only:** You can also install the upstream release from npm, but it will **not** work on Homebridge v2:
> ```shell
> npm install -g homebridge-nest
> ```

---

### Configuration

Add a platform entry to your `~/.homebridge/config.json`:

```json
"platforms": [
  {
    "platform": "Nest",
    "options": [ "HomeAway.Disable" ],
    "access_token": "your Nest Account access token"
  }
]
```

**Required fields — Nest Account (access token):**
- `"platform"`: Must always be `"Nest"`
- `"access_token"`: Nest service access token

**Required fields — Google Account (cookies method):**
- `"platform"`: Must always be `"Nest"`
- `"googleAuth"`: Google authentication object — see [Using a Google Account](#using-a-google-account) below

**Optional fields:**
- `"structureId"`: Filter to a specific structure/home (see logs on first run)
- `"options"`: Array of feature flags — see [Feature Options](#feature-options)
- `"fanDurationMinutes"`: Minutes to run the fan when turned on manually (default: `15`)
- `"hotWaterDurationMinutes"`: Minutes to run hot water when turned on manually (default: `30`, UK/EU only)

---

### Using a Nest Account

Go to `https://home.nest.com` in your browser and log in. Then navigate to `https://home.nest.com/session`. You will see a JSON response like:

```json
{"2fa_state":"not_enrolled","access_token":"XXX","email":"...","expires_in":"...", ...}
```

Copy the value of `access_token` (the `XXX` — a long string beginning with `b`) and set it in your `config.json`. **Do not log out of `home.nest.com`**, as this will invalidate your token — just close the tab.

---

### Using a Google Account

Google Accounts are configured using a `"googleAuth"` object in `config.json`:

```json
"platform": "Nest",
"googleAuth": {
  "issueToken": "https://accounts.google.com/o/oauth2/iframerpc?action=issueToken...",
  "cookies": "OCAK=TOMPYI3cCPAt...; SID=ogftnk...; HSID=ApXSR...; ...; SIDCC=AN0-TYt..."
}
```

To obtain your `issueToken` and `cookies` values (this only needs to be done once):

#### Chrome

1. Open a Chrome tab in **Incognito Mode** (or clear your cache).
2. Open **Developer Tools** → **Network** tab. Enable **Preserve Log**.
3. Filter by `issueToken`.
4. Go to `home.nest.com`, enable third-party cookies (eye icon in the address bar), then click **Sign in with Google**.
5. Click the `iframerpc` network call that appears. Under **Headers → General**, copy the full **Request URL** — this is your `"issueToken"`.
6. Change the filter to `oauth2/iframe`. Click the last `iframe` call.
7. Under **Headers → Request Headers**, copy the entire `cookie` value (the whole multi-line string, without the `cookie:` label) — this is your `"cookies"`.
8. Close the tab. **Do not log out.**

#### Safari

1. Open a Safari tab in **Private Mode**.
2. Open **Developer Tools** → **Network** tab. Enable **Preserve Log**.
3. Filter by `issueToken`.
4. Go to `home.nest.com` and click **Sign in with Google**.
5. Click the `iframerpc` network call. Under **Headers → Summary**, copy the URL — this is your `"issueToken"`.
<img width="762" alt="Screenshot 2024-10-30 at 09 14 38" src="https://github.com/user-attachments/assets/0e65b5a3-018a-4ca0-8bac-e3c4c52016a0">

6. Change the filter to `oauth2/iframe`. Click the most recent `iframe` call.
7. Under **Headers → Request**, copy the entire `cookie` value — this is your `"cookies"`.
<img width="762" alt="Screenshot 2024-10-30 at 09 22 15" src="https://github.com/user-attachments/assets/c802201b-463c-4e78-9fa9-f2ed4f60fffe">

8. Close the tab. **Do not log out.**

---

### HomeKit Accessories

#### Home

- *Switch* — **Home Occupied**: reflects and controls the Home/Away state

#### Nest Thermostat (+ Temperature Sensors)

- *Thermostat* — ambient temperature & humidity, mode control (heat/cool/auto/off), target temperature
- *Switch* — **Eco Mode**: turn Eco Mode on/off
- *Fan* — fan control (not available on EU/UK Thermostat E)
- *TemperatureSensor* — thermostat built-in temperature (disabled by default when no separate sensors are present)
- *TemperatureSensor* — one per additional Nest Temperature Sensor
- *HumiditySensor* — built-in humidity (disabled by default)

#### Nest Protect

- *SmokeSensor* — smoke detected
- *CarbonMonoxideSensor* — CO detected
- *OccupancySensor* — occupancy/motion (wired Protects only)

#### Nest x Yale Lock

- *LockMechanism*

---

### Feature Options

Set `"options"` in `config.json` to an array of strings chosen from the following:

| Option | Effect |
|---|---|
| `"Thermostat.Disable"` | Exclude all Nest Thermostats |
| `"Thermostat.Fan.Disable"` | No Fan accessory |
| `"Thermostat.Eco.Disable"` | No Eco Mode switch |
| `"Thermostat.SeparateBuiltInTemperatureSensor.Enable"` | Add a separate temperature sensor for the thermostat |
| `"Thermostat.SeparateBuiltInHumiditySensor.Enable"` | Add a separate humidity sensor for the thermostat |
| `"Thermostat.EcoMode.ChangeEcoBands.Enable"` | Changing temp in Eco Mode adjusts Eco bands instead of exiting Eco Mode |
| `"TempSensor.Disable"` | Exclude Nest Temperature Sensors |
| `"HomeAway.Disable"` | Exclude Home/Away switch |
| `"HomeAway.AsOccupancySensor"` | Home/Away as OccupancySensor instead of Switch |
| `"HomeAway.AsOccupancySensorAndSwitch"` | Home/Away as both OccupancySensor and Switch |
| `"Protect.Disable"` | Exclude Nest Protects |
| `"Protect.MotionSensor.Disable"` | No motion/occupancy sensor for Protects |
| `"Lock.Disable"` | Exclude Nest x Yale Locks |
| `"Nest.FieldTest.Enable"` | Use Nest Field Test account (experimental) |

To apply an option to one specific device, append its `device_id` (shown in logs or as *Serial Number* in HomeKit): e.g. `"Thermostat.Disable.09AC01AC31180349"`.

---

### Things to Try with Siri

- *Hey Siri, set the temperature to 72 degrees.*
- *Hey Siri, set the temperature range to between 65 and 70 degrees.*
- *Hey Siri, set the thermostat to cool.*
- *Hey Siri, turn on the air conditioning.*
- *Hey Siri, turn Eco Mode on.*
- *Hey Siri, what's the temperature at home?*
- *Hey Siri, what's the temperature in the Basement?*
- *Hey Siri, what's the status of my smoke detector?*
- *Hey Siri, unlock my Front Door.*

---

### Credits

This plugin is a fork of [homebridge-nest](https://github.com/chrisjshull/homebridge-nest) by [chrisjshull](https://github.com/chrisjshull) and [adriancable](https://github.com/adriancable). If you find the original project useful, consider [supporting it](https://paypal.me/adriancable586).

If you want a plug-and-play Nest integration without running Homebridge, check out [Starling Home Hub](https://www.starlinghome.io).
