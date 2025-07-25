# Lightspeed to Dataverse Exporter

A service that exports Lightspeed data to Dataverse for analysis and storage. It periodically scans for JSON data files, packages them, and sends them to the Red Hat.

## Quick Start

### Installation

```bash
# Install dependencies
uv sync
```

### Configuration

You can either use `config.yaml` or provide values as command-line arguments.
When both are provided, command-line arguments take precedence over config.yaml values.

**Option 1: Using config.yaml**

Create a `config.yaml` file. See example: [config.yaml.example](config.yaml.example)

```yaml
data_dir: "/path/to/data"
service_id: "your-service-id"
ingress_server_url: "https://console.redhat.com/api/ingress/v1/upload"
collection_interval: 3600  # 1 hour
cleanup_after_send: true
```

**Option 2: Using command-line arguments only**

```bash
uv run python -m src.main \
  --data-dir /path/to/data \
  --service-id <your-service-id> \
  --ingress-server-url "https://console.redhat.com/api/ingress/v1/upload" \
  --collection-interval 3600 \
  --mode manual \
  --ingress-server-auth-token your-token \
  --identity-id your-identity
```

**Option 3: Combining both (args override config)**

```bash
# config.yaml has most settings, but override specific values
uv run python -m src.main \
  --config config.yaml \
  --log-level DEBUG \
  --collection-interval 60  # Override the config.yaml value
```

### Usage

The service supports two authentication modes depending on your deployment environment:

**OpenShift Mode** (recommended for cluster deployments):
```bash
uv run python -m src.main --mode openshift --config config.yaml
```
- Automatically retrieves auth token from cluster pull-secret
- Gets identity ID (cluster ID) from cluster version
- No manual token management required

**Manual Mode** (for local testing or non-OpenShift environments):
```bash
uv run python -m src.main --mode manual --config config.yaml \
  --ingress-server-auth-token YOUR_TOKEN \
  --identity-id YOUR_IDENTITY
```
- Requires explicit credentials
- Use `scripts/ingress_token_from_offline_token.py` to generate tokens for stage testing

### Common Options

```bash
# Run with debug logging
uv run python -m src.main --config config.yaml --log-level DEBUG

# Run once without continuous collection
uv run python -m src.main --config config.yaml --collection-interval 0

# Keep files after sending (useful for testing)
uv run python -m src.main --config config.yaml --no-cleanup
```

## Data Format

The service scans for JSON files in these subdirectories under your configured `data_dir`:

- `feedback/` - User feedback data
- `transcripts/` - Conversation transcripts

Example structure:
```
data/
├── feedback/
│   ├── feedback1.json
│   └── feedback2.json
└── transcripts/
    ├── conversation1.json
    └── conversation2.json
```

**Note:** Files in other directories are ignored. The service recursively scans for `*.json` files only.

## Container Deployment

### Build and Run Locally

```bash
# Build container
make build

# Run with config and data mounted
podman run --rm \
  -v ./config.yaml:/config/config.yaml \
  -v ./data:/data \
  lightspeed-exporter --config /config/config.yaml
```

## Documentation

- **[ONBOARDING.md](ONBOARDING.md)** - Complete setup and testing guide
- **[examples/README.md](examples/README.md)** - Kubernetes deployment examples

