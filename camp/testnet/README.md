## Running Nodes for Camp with Docker

#### Create a jwt token

```sh
openssl rand -hex 32 | tr -d "\n" > "./artifacts/jwt.hex"
```

#### Running ABC Node (Consensus Node)

Then start the ABC consensus node, connecting to the XYZ node:

```sh
# Create data directory
mkdir -p .abc

# Run ABC consensus layer node
docker run -d \
  --name abc \
  --network host \
  -v $(pwd)/.abc:/data \
  -v $(pwd)/artifacts/jwt.hex:/jwt.hex \
  -p 8545:8545 \
  -p 9000:9000 \
  ghcr.io/gelatodigital/abc node \
  --datadir /data \
  --execution.jwtsecret /jwt.hex \
  --execution.endpoint "http://0.0.0.0:8451" \
  --back-sync.rpc https://rpc.basecamp.t.raas.gelato.cloud \
  --http \
  --log.format json \
  --feed.ingress wss://feed.basecamp.t.raas.gelato.cloud \
  --sequencer.pubkey 38d7bc04511f874ee18935e09284384c1b85513bdb975799d280b626fd86e5f7
  --da.endpoint public-celestia-mocha4-consensus.numia.xyz:26657
```


#### Running XYZ Node (Execution Node)

Start the XYZ execution node first:

```sh
# Create data directory
mkdir -p .xyz

# Run XYZ execution layer node
docker run -d \
  --name xyz \
  --network host \
  -v $(pwd)/.xyz:/data \
  -v $(pwd)/artifacts/jwt.hex:/jwt.hex \
  -v $(pwd)/artifacts/genesis.json:/genesis.json \
  -p 8449:8449 \
  -p 8450:8450 \
  -p 8451:8451 \
  -p 30303:30303 \
  ghcr.io/gelatodigital/xyz node \
  --chain /genesis.json \
  --datadir /data \
  --http \
  --http.api "eth,net,web3,debug,txpool" \
  --http.addr "0.0.0.0" \
  --http.port "8449" \
  --ws \
  --ws.addr "0.0.0.0" \
  --ws.port "8450" \
  --authrpc.addr "0.0.0.0" \
  --authrpc.port "8451" \
  --authrpc.jwtsecret /jwt.hex \
  --log.stdout.format=json \
  --log.file.max-files=0 \
  --disable-discovery \
  --trusted-only \
  --trusted-peers enode://7a4fde5f5eb926767730dace9f99e10a4060d5c62fc82a27b42a4684d99422f0bb80d418d1c524587d67e21c24a064c775f61ca0b92ebf5fb3aa172a9eda1c2a@34.40.24.94:30303 \
  --abc.sequencer-http https://rpc.basecamp.t.raas.gelato.cloud
```
