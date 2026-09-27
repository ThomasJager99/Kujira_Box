<p align="center">
  <img src="../../../assets/logos/firefly-iii.svg" width="180">
</p>

<br>

This directory contains an example deployment of Firefly III, an open-source personal finance manager used to track money, budgets and accounts. Firefly III provides a private, self-hosted web interface where you record income and expenses, split them across accounts, set budgets and category rules, and read back where your money actually goes through reports and charts. Unlike cloud finance apps, every transaction stays on your own server — the application talks only to its own database and never phones a third party unless you explicitly connect one.

## Official container image

https://github.com/firefly-iii/firefly-iii — image published as `fireflyiii/core`.

## Deployment Notes

- Runs as three containers: the Firefly III web application, a MariaDB database that holds all financial data, and a small Alpine cron container that pings the app once a day so scheduled jobs (recurring transactions, bill reminders) actually fire.
- Stores database data and uploaded attachments in persistent Docker volumes (`firefly_db`, `firefly_upload`) that survive restarts and image upgrades.
- Exposes a single HTTP interface; intended to sit behind a reverse proxy (Caddy / Traefik / nginx) for TLS in production.
- Includes healthchecks for all three containers, and the app only starts once the database reports healthy, so the first-run migrations never race an unready database.
- Memory limits and reservations are set on every container to keep the stack polite on a shared homelab host.
- Environment-specific values (host port, memory limits, database credentials, `APP_KEY`, timezone, cron token) are externalized to a `.env` file — nothing sensitive lives in the compose file.
- On first start Firefly runs its own database migrations automatically; no SQL import is needed. Register the first account, then disable further registration in the admin settings.

## Example Use

Firefly III can be used to organize and track:

- personal day-to-day income and expense tracking
- monthly budgets with category rules and automatic classification
- multiple accounts (bank, cash, credit, savings) and overall net worth
- bills, subscriptions and recurring-transaction reminders
- financial reporting — spending by category, over time, and account balances

This lets an individual or household manage money privately through a fully self-hosted alternative to cloud budgeting apps.

## Bank import (optional)

Firefly III does not connect to banks itself, by design. To import transactions, run the separate [Data Importer](https://github.com/firefly-iii/data-importer): it supports CSV files (fully local, nothing leaves your server) and, for EU banks, GoCardless/Nordigen.

## Thanks

Thanks to the Firefly III project and its maintainers for providing a powerful, genuinely private, open-source personal finance manager suitable for self-hosted environments.

## Links

- Website: https://www.firefly-iii.org
- Source: https://github.com/firefly-iii/firefly-iii
- Documentation: https://docs.firefly-iii.org
- Data Importer: https://github.com/firefly-iii/data-importer
