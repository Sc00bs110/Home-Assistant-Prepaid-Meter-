# Home-Assistant-Prepaid-Meter-
Some YAML code to monitor the balance on my prepaid meter using the Inverter Total Grid Import (kWh) sensor from my inverter to monitor power usage

## What this repository contains
- `prepaid_meter.yaml` — Home Assistant package: inputs, sensors and automations (place under `config/packages/`).
- `prepaid_dashboard.yaml` — Lovelace dashboard (YAML): dashboard used by the package (place under `config/packages/`).
- `Homeassistant_include_in_Configuration.yaml` — contains a `homeassistant:` snippet to enable the `packages` folder include (paste into your `configuration.yaml` if not already present).

**Important entity names used by the package** (you will see these in Home Assistant):
- `input_number.prepaid_balance` — Set the prepaid kWh amount when you recharge (this is the starting balance).
- `input_number.prepaid_grid_import_reference` — Stores the inverter total grid import reading at the moment you recharge.
- `input_datetime.prepaid_last_recharge` — Timestamp of the last recorded recharge.
- `sensor.prepaid_grid_import_reference_display` — Read-only display of the grid import reference (kWh).
- `sensor.prepaid_energy_used_since_recharge` — `current inverter total import - reference` (kWh used since recharge).
- `sensor.prepaid_balance_current` — Current running prepaid balance (kWh remaining).
- `input_number.prepaid_low_balance_threshold` — Dashboard-settable low-balance alert threshold (kWh).
- `sensor.inverter_total_grid_import_kwh` — **Your inverter-provided sensor** (must exist; the package reads this to compute usage).

---

## Prerequisites (what you need first)
1. A running **Home Assistant** instance (Supervised / Home Assistant OS / Container / Core).  
2. Access to your Home Assistant `config` folder (via File Editor add-on, Samba, or SSH).  
3. Your inverter already integrated in Home Assistant so that `sensor.inverter_total_grid_import_kwh` exists and updates.  

Optional 
4. A notification method configured in Home Assistant (Mobile App, Email/SMTP, or similar).

---

## Installation — step by step

> **Step 0 — Backup**: Before making any changes, make a backup of your Home Assistant config folder (copy the folder or use the snapshot feature).

### Step 1 — Enable packages (if not already)
1. Open your `configuration.yaml` (Menu → Settings → Add-ons → File editor, or edit via Samba/SSH).
2. Ensure you have the following **exact** block (it tells Home Assistant to load files from `config/packages/`):

```yaml
homeassistant:
  packages: !include_dir_named packages
```

3. If you already use `homeassistant:` settings in `configuration.yaml`, **merge** the `packages:` line into the existing block instead of duplicating `homeassistant:`. (If you are unsure, paste the exact snippet above into your config and check configuration in step 4.)

### Step 2 — Create the packages folder
If you don't already have a `packages` folder in your configuration directory, create it:

```
/config/packages/
```

### Step 3 — Copy the YAML files
Place the following files into the `config/packages/` folder:

- `prepaid_meter.yaml` → `/config/packages/prepaid_meter.yaml`
- `prepaid_dashboard.yaml` → `/config/packages/prepaid_dashboard.yaml`

> Note: the `prepaid_meter.yaml` package references the dashboard filename `packages/prepaid_dashboard.yaml` (this is why the dashboard also lives in the `packages` folder).

### Step 4 — Update notification service names (important)
Open `/config/packages/prepaid_meter.yaml` and look for the **low-balance notification** automation. By default the automation uses `notify.notify` as a placeholder. Replace that placeholder with your actual notification services:

- Mobile app push: `notify.mobile_app_<your_device_name>` (check *Developer Tools → Services* to discover exact name)
- Email: whatever service you've configured for email (for example `notify.smtp` or `notify.mail` — depends on your setup)
- The package already creates a persistent dashboard notification (`persistent_notification.create`) so no extra service is required for that.

Example action block (replace service names with your actual services):

```yaml
action:
  - service: notify.mobile_app_johns_phone
    data:
      title: "⚠️ Prepaid Low Balance"
      message: "Your prepaid balance is {{ states('sensor.prepaid_balance_current') }} kWh — below threshold of {{ states('input_number.prepaid_low_balance_threshold') }} kWh."
  - service: notify.notify   # optional: your general notify service
    data:
      title: "Prepaid Low Balance"
      message: "Balance {{ states('sensor.prepaid_balance_current') }} kWh — recharge soon."
  - service: persistent_notification.create
    data:
      title: "Low Prepaid Balance"
      message: "Your prepaid balance is **{{ states('sensor.prepaid_balance_current') }} kWh**. Please recharge."
      notification_id: "low_prepaid_balance_alert"
  # Optional: email
  - service: notify.smtp
    data:
      title: "Low Prepaid Balance"
      message: "Prepaid balance now {{ states('sensor.prepaid_balance_current') }} kWh."
      target: "you@example.com"
```

> If you don’t want email, remove that block. If you don’t have the mobile app, remove the mobile service line. Use Developer Tools → Services to discover and test notify services.

### Step 5 — Validate configuration and restart Home Assistant
1. In Home Assistant go to **Settings → System → Server Controls**.  
2. Click **Check configuration** (fix any YAML errors reported).  
3. Click **Restart** (Home Assistant will restart and load the new package).

### Step 6 — Use the dashboard (how to operate)
1. In Home Assistant's sidebar you should now see **Prepaid Meter** (if `show_in_sidebar: true` is set in the package).  
2. Open the dashboard. The important controls are:
   - **Set Prepaid Balance (kWh)** — Enter the kWh you purchased when you recharge. This must be set manually at the time of recharge. The system will **record the inverter reading** at that exact moment and store the timestamp.  
   - **Last Recharge Date/Time** — Shows when you last recorded the balance.  
   - **Grid Import Reference at Recharge** — Read-only display of the inverter total grid import at the time of recharge.  
   - **Energy Used Since Recharge** — Calculated as `sensor.inverter_total_grid_import_kwh - input_number.prepaid_grid_import_reference`.  
   - **Current Balance** — Shows your remaining prepaid kWh (`input_number.prepaid_balance - energy_used_since_recharge`).  
   - **Low Balance Threshold** — Set the kWh level at which you want to be notified (edit on dashboard).

### Step 7 — Test it
1. Set `input_number.prepaid_low_balance_threshold` to a value above your current `sensor.prepaid_balance_current`. This should trigger the low-balance automation quickly (you can set it high temporarily for testing).  
2. Check that you receive:
   - a mobile push (if configured),  
   - a persistent dashboard notification (always created), and  
   - an email (if configured).  
3. Reset the threshold to your normal desired alert level after testing.

---

## How the system works (short)
1. You set the **starting prepaid kWh** in `input_number.prepaid_balance` when you recharge.  
2. An automation records the current `sensor.inverter_total_grid_import_kwh` into `input_number.prepaid_grid_import_reference` and stores the timestamp in `input_datetime.prepaid_last_recharge`.  
3. `sensor.prepaid_energy_used_since_recharge` computes how many kWh you’ve used since recharge (never negative).  
4. `sensor.prepaid_balance_current` subtracts the used amount from your starting `input_number.prepaid_balance` to give the running remaining kWh.  
5. The low-balance automation compares `sensor.prepaid_balance_current` with the user-set `input_number.prepaid_low_balance_threshold` and sends notifications when the balance drops below the threshold. A persistent notification is also created on the dashboard (and dismissed automatically when the balance rises again).

---

## Troubleshooting & FAQ

**Q — I changed the balance and after restart it reverted or changed automatically.**  
A — Check if you have an automation that updates `input_number.prepaid_balance` automatically. The package included with these instructions should **not** overwrite `input_number.prepaid_balance` on reboot. If you still see overwrites, search your automations for any `input_number.set_value` calls targeting `input_number.prepaid_balance` and disable/remove them.

**Q — The dashboard doesn’t appear / cards are missing.**  
A — If you use the UI (storage-based) Lovelace mode rather than YAML mode, the `lovelace:` dashboard YAML may not load. You can either switch to YAML mode or manually recreate the cards using the Dashboard editor and copy the entities from this package.

**Q — I don’t receive mobile notifications.**  
A — Confirm your mobile app integration is installed and you used the exact notify service name (Developer Tools → Services shows available `notify.*` services). Test notifications directly in Developer Tools → Services with a small payload first.

**Q — I don’t receive email notifications.**  
A — Make sure you have an email/SMTP notify integration configured correctly. Email tools vary—consult Home Assistant docs for `notify.smtp` or other email add-ons.

**Q — I see wrong values (negative balance, weird numbers).**  
A — Ensure `sensor.inverter_total_grid_import_kwh` is the **cumulative** grid-import kWh meter (not an instant power sensor). The template assumes a monotonically increasing total kWh number. If your inverter provides energy in Wh or another unit, convert accordingly (edit the template sensors in `prepaid_meter.yaml`).

---

## Advanced (optional)
- Add an `input_boolean.prepaid_alert_sent` and modify the automation so you only get one notification until the balance rises above the threshold again (helps avoid repeated alerts).  
- Log recharges to a CSV or use the Recorder/History to chart prepaid balance over time.

---

## Final notes
- This package is intended to be **minimal and safe** — it uses persistent `input_number` and `input_datetime` entities so values survive restarts.  
- Always test notification changes carefully.  
