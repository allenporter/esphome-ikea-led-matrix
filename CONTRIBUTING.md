# Development

## Preparing Environment

Set up a Python virtual environment and install the development dependencies:

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements_dev.txt
```

Or using [`uv`](https://docs.astral.sh/uv/):

```bash
uv venv
source .venv/bin/activate
uv pip install -r requirements_dev.txt
```

## Validating and Compiling Test Configurations

Verify and compile the test configuration against the local component:

```bash
esphome config tests/frekvens.yaml
esphome compile tests/frekvens.yaml
```
