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

**Optional **
4. A notification method configured in Home Assistant (Mobile App, Email/SMTP, or similar).



Step-by-step installation
Backup first
Create a snapshot or copy your /config folder.

Create the packages folder
If it doesn’t exist: /config/packages/

Create the package file
Create /config/packages/prepaid_meter.yaml and paste the entire package YAML from section 1 above.

Enable packages in configuration.yaml
Open /config/configuration.yaml and make sure you have:

yaml
Copy
Edit
homeassistant:
  packages: !include_dir_named packages
If homeassistant: already exists, merge the packages: line under it (don’t duplicate the top-level key).

Check your inverter sensor name
The package expects sensor.inverter_total_grid_import_kwh (a cumulative import reading).
If yours is different, search/replace the name in the package YAML.

Set your notification services
In the package, find the automation “Prepaid: Low Balance Alert” and replace:

notify.mobile_app_YOURDEVICE → with your mobile app service (Developer Tools → Services → look for notify.mobile_app_*).

Leave notify.notify as a generic fallback, or remove it if not needed.

Optional: uncomment the notify.smtp block if you have email configured.

Restart Home Assistant
Go to Settings → System → Server Controls → Check configuration → Restart.

Create the dashboard

Go to Settings → Dashboards → + Add Dashboard.

Name it “Prepaid Meter”, open it, click ⋮ → Raw configuration editor.

Paste the dashboard YAML from section 2, then Save.

Use it

On the dashboard, enter the Set Prepaid Balance (kWh) to match your meter after you purchase credit.

Click Recharge Now → this captures your inverter’s total import as the reference and timestamps the recharge.

The Status card shows Energy Used, Current Balance, and lets you set the Low Balance Alert Threshold (persistent).

You’ll get alerts when Current Balance drops below your threshold.

Test the alert
Temporarily set Low Balance Alert Threshold higher than your current balance to force a notification, then set it back.

