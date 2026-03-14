# Profistranka

Multi‑tenant CMS for AI‑generated websites.

## What is Profistranka?

Profistranka lets you deploy a complete website (HTML, CSS, JS, images) with a single API call. Each website gets:
- A unique subdomain (`*.profistranka.cz`)
- Automatic HTTPS (Let’s Encrypt)
- An admin dashboard to edit content
- API access for programmatic updates

## Quick start for AI agents

If you’re an AI agent (like ChatGPT, Claude, or a custom AI) and want to upload a website you’ve generated, read:

**[agents.md](agents.md)** – step‑by‑step guide with code examples.

## For developers

The full source code of the Profistranka CMS is in a separate repository:  
[https://github.com/Matkalcz/profistranka‑system](https://github.com/Matkalcz/profistranka-system)

This repository (`Matkalcz/profistranka`) contains only the **AI‑agent documentation** and high‑level project description.

## Features

- **One‑click website creation** via REST API
- **Automatic SSL** for every subdomain
- **File‑based content management** – upload HTML, CSS, JS, images
- **Live editor** – edit files directly in the admin dashboard
- **Custom domain support** – attach your own domain with auto‑SSL
- **View statistics** – track page views per website
- **Docker‑based deployment** – easy to host yourself

## API endpoint

Base URL: `https://admin.profistranka.cz/api/v1`

See [agents.md](agents.md) for detailed API usage.

## License

MIT