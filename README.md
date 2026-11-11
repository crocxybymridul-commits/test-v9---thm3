# PX9V Landing Provisioner — updated template

This package contains a Telegram landing/store bot that provisions isolated PX9V bot instances from the supplied `9V-main.zip` template.

## Landing flow

`/start` → **BUY NEW BOT** → send access/create key → send customer bot token → send Admin ID → bot is provisioned and started.

If the access key was created with Premium enabled, the new PX9V instance receives a one-time PX9V-compatible Premium Key. The key uses the same `premium_keys` schema and redemption flow as the supplied PX9V source.

## Admin

`/admin` opens the landing admin panel.

**CREATE TOKEN** asks for:
1. USER / bot-creation count
2. Whether a Premium Key should be included
3. Premium KEY USER limit
4. Premium KEY REVENUE limit
5. Premium KEY TIME in days

The access key can therefore provision exactly the configured number of customer bots.

## Revoke

A customer can revoke their own bot with:

- `/revoke @botusername`
- `/revoke ACCESS_TOKEN`

Revoking an access token also revokes/stops every bot created from that access token.

The landing admin can revoke any bot/token.

## Important environment variables

```text
LANDING_BOT_TOKEN=...
LANDING_ADMIN_ID=...
LANDING_DB_PATH=/data/landing_bot.db
INSTANCE_ROOT=/data/instances
TEMPLATE_ROOT=px9v_template
MASTER_INSTANCE_KEY=<44-character URL-safe Fernet key>
REVOKE_DELETE_FILES=true
```

Generate `MASTER_INSTANCE_KEY` without installing cryptography on the phone:

```bash
python -c "import secrets,base64; print(base64.urlsafe_b64encode(secrets.token_bytes(32)).decode())"
```

## Railway

Add a Railway Volume mounted at `/data`. This keeps the landing DB and customer bot instances across redeployments.

Start command:

```text
python main.py
```

## PX9V template / QR handling

The customer bot template is the newly supplied `9V-main.zip`, not the previous template.

Its `requirements.txt` includes `qrcode[pil]>=8.0`. Additionally, the QR code handler has an HTTP QR fallback, so the customer bot does not show the old `QR module is missing on the server` error merely because the optional local QR package is unavailable.


## New son-bot permissions and themes

- Every ID in `LANDING_ADMIN_ID` and the optional comma-separated `LANDING_ADMIN_IDS` is passed into each provisioned son bot as `FATHER_ADMIN_IDS`.
- Those father-bot administrators receive `/padmin` and the same permanent-admin permissions inside every son bot. The customer-selected Admin ID remains a permanent admin too.
- Son-bot `/admin` includes **CHANGE THEME**, **WELCOME VIDEOS**, and **RESTRICT BOT**.
- Theme 1 preserves the existing layout. Themes 2 and 3 show every premium plan directly on the main menu, one button per row, followed by menu buttons in pairs (Menu 1 | Menu 2, then Menu 3, etc.).
- Theme 3 additionally sends up to five admin-configured welcome videos as one Telegram media album, followed by the saved welcome message and its main-menu buttons. Configure them from `/admin` → **WELCOME VIDEOS**. Send videos one by one, then press **DONE**. **CLEAR ALL** removes the saved list.
- `/buttoncolor [ID] [R/G/B]` controls premium-button colors in the currently selected theme; `/menucolor [ID] [R/G/B]` controls menu-button colors.

## Protected content limitation

**RESTRICT BOT** applies Telegram's `protect_content` flag to newly sent/copied bot content while enabled, including welcome albums, welcome copies, premium menus, QR images, and broadcast copies. The toggle takes effect immediately for supported sends; previously sent messages are not retroactively protected. Telegram bots cannot prevent screenshots, screen recordings, or external capture.
