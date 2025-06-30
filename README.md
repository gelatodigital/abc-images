# ABC - Abundance Sovereign Stack

### Prerequisites

- GitHub Personal Access Token (PAT)
- Install [Docker](https://docs.docker.com/get-docker/)

## Restricted Access

### Generate a GitHub Personal Access Token (PAT)
	1.	Go to https://github.com/settings/tokens
	2.	Click “Generate new token (classic)” (or use fine-grained tokens if preferred)
	3.	Give it a name and expiration date
	4.	Under Scopes, select:
	    •	read:packages (required to pull images)
	    •	repo
	5.	Click Generate token
	6.	Copy and save the token somewhere safe

### Login to GHCR via Docker
```bash
echo <YOUR_PAT> | docker login ghcr.io -u <GITHUB_USERNAME> --password-stdin
```
Replace:
	•	<YOUR_PAT> with your GitHub personal access token
	•	<GITHUB_USERNAME> with your GitHub username

### Pull the images

```bash
docker pull ghcr.io/gelatodigital/abc
docker pull ghcr.io/gelatodigital/xyz
```


## Running with Docker

#### Running ABC Node (Consensus Node)

```sh
# Create data directory
mkdir -p .tmp/abc

# Run ABC consensus layer node
docker run -d \
  --name abc \
  -v $(pwd)/.tmp/abc:/data \
  -v $(pwd)/artifacts/jwt.hex:/jwt.hex \
  -p 8545:8545 \
  -p 9000:9000 \
  ghcr.io/gelatodigital/abc node \
  --datadir /data \
  --execution.jwtsecret /jwt.hex \
  --execution.endpoint "http://0.0.0.0:8451" \
  --http \
  --log.format json \
  --feed.ingress ${FEED_URL} \
  --sequencer.pubkey ${SEQUENCER_PUBLIC_KEY}
```

#### Running XYZ Node (Execution Node)

```sh
# Create data directory
mkdir -p .tmp/xyz

# Run XYZ execution layer node
docker run -d \
  --name xyz \
  -v $(pwd)/.tmp/xyz:/data \
  -v $(pwd)/artifacts/jwt.hex:/jwt.hex \
  -v $(pwd)/artifacts/genesis.json:/genesis.json \
  -p 8449:8449 \
  -p 8450:8450 \
  -p 8451:8451 \
  ghcr.io/gelatodigital/xyz node \
  --chain /genesis.json \
  --datadir /data \
  --http \
  --http.api "eth,net,web3,debug,txpool" \
  --http.addr "0.0.0.0" \
  --http.port "8449" \
  --ws \
  --ws.addr "0.0.0.0" \
  --ws.port "8450 \
  --http.api eth,net,admin \
  --authrpc.addr "0.0.0.0" \
  --authrpc.port "8451"
  --authrpc.jwtsecret /jwt.hex \
  --log.stdout.format=json \
  --log.file.max-files=0 \
  --disable-discovery \
  --trusted-only \
  --trusted-peers ${SEQUENCER_ENODE} \
  --abc.sequencer-http ${GELATO_PUBLIC_ENDPOINT}
```

Please note that all arguments marked with `${}`must be replaced accordingly.