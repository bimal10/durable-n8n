# durable-n8n

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

All of my n8n workflows

## 🔄 Automated Backup Setup

This repository includes a native n8n workflow designed to automatically back up all of your n8n workflows directly into a dedicated folder. 

To ensure clean preservation of your data, the backup JSON files are written out to a dedicated backups folder within the n8n workspace.

Because Docker container directories are cleanly mapped to your Mac directories via volume mounts, any workflow backups made by n8n will immediately appear in your local `backups/` directory on Mac without needing to extract them manually or use commands like `docker cp`.

> **Note on Version Control:** By default, the `backups/` folder is included in the `.gitignore` file to prevent inadvertently publishing sensitive workflows to a public repository. If you fork this project to a *private* Git repository and want to version control your workflows automatically, **remember to remove `backups/` from your `.gitignore` file** so Git tracks them!

### 🐳 Running via Local Docker-Compose (Devcontainer)

If you are using the devcontainer deployment (`/path/to/your/n8n/.devcontainer/docker-compose.yml`), it already sets up an underlying PostgreSQL database and correctly maps your workflows automatically via Docker volumes:

```yaml
services:
  n8n:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ..:/workspaces:cached
      - /path/to/your/durable-n8n/backups:/home/node/.n8n/backups
    environment:
      DB_POSTGRESDB_HOST: postgres
      DB_TYPE: postgresdb
```

*(Note: If your workflows are failing to write because the path isn't strictly identical to the host path, use the Docker CLI command below instead which explicitly symmetrically maps the directory and passes the `N8N_RESTRICT_FILE_ACCESS_TO` variable).*

### 🐳 Running n8n via Docker CLI

Alternatively, you can securely start the n8n container with Docker CLI. Use the following command. This enforces symmetrical path mapping and explicitly grants the container write access to your local Mac directory to completely avoid any write permission issues:

```bash
docker run -it --rm --name n8n -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  -v /path/to/your/durable-n8n/backups:/path/to/your/durable-n8n/backups \
  -e N8N_RESTRICT_FILE_ACCESS_TO=/path/to/your/durable-n8n/backups/ \
  docker.n8n.io/n8nio/n8n
```

The workflow file is located at [`backup-workflows-to-repo.json`](./backup-workflows-to-repo.json).

### Steps to Run

1. Open your n8n interface.
2. In the top-right corner, click **Options (the 3 vertical dots) -> Import from File**.
3. Select the `backup-workflows-to-repo.json` file.
4. **Important:** Open the **Write File to Disk2** node and update the `/path/to/your/durable-n8n/backups/` string in the "File Name" parameter to match the actual absolute path to your backups folder.
5. Open the **Get All Workflows** (`n8n` API) node.
6. In the **Credential** dropdown, click "Create New Credential".
7. Follow the "Setup n8n API Key" instructions below to authenticate the node.
8. Save your workflow and at the top right, toggle it **Active**. This ensures the scheduled backup triggers run automatically every day.
9. (Optional) Run the workflow manually once by clicking the "Execute Workflow" button at the bottom to trigger the initial backup.

### Setting up the n8n API Key

In order to allow your n8n workflows to access your other flows securely, you must configure n8n's public API credentials.

1. Navigate to your n8n application **Settings** (bottom left corner of the UI menu).
2. Click on **n8n API** in the settings sidebar.
3. If this is your first time, you may need to enable the Public API.
4. Click the **Create an API key** button and give it a valid name (e.g. `Backup Workflow Key`).
5. Carefully copy the generated API Key.
6. Return to your backup workflow, create a new "n8n API" credential inside the **Get All Workflows** node.
7. Paste the API Key there, and for the **Base URL**, use the exact same URL you use to access your n8n interface in your browser, followed by `/api/v1`. (e.g., `http://localhost:5678/api/v1` or `https://n8n.your-domain.com/api/v1`).
8. Your n8n API Node can now successfully fetch and archive all of your workflows!

## ⚠️ Limitations

Currently, this backup workflow is only supported on **self-hosted, local systems** (such as Docker or local installations). This is because the workflow relies on writing files directly to the local disk using the `n8n-nodes-base.readWriteFile` node, which requires access to the host machine's file system through mapped volumes.

It will **not** work correctly on n8n Cloud or restricted remote environments without additional modifications (like changing the node to upload to external storage instead of a local disk).

## 🤝 Contributing

Contributions, issues and feature requests are welcome! Feel free to check the issues page or submit a pull request.

## 📄 License

This project is [MIT](LICENSE) licensed.

## 📫 Contact

Created by Bimal Gupta

- **GitHub:** [@bimalgupta](https://github.com/bimalgupta)
