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
  --http \
  --log.format json \
  --feed.ingress wss://feed.camp.raas.gelato.cloud \
  --sequencer.pubkey 62fbcb55243d1523595aa26d9ca95cbbbb25280bf17d83172792d2a597e874ab \
  --da.endpoint rpc.celestia.pops.one:26657
```

Notes: Please replace `--da.endpoint` accordingly, the one provided above is only used as an example. Do not rely on the endpoint above for production deployments. We strongly advise to use a production ready endpoint or run your own Celestia light node.


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
  --trusted-peers enode://844d50728ccd7e4411d29c6e14642ce65c42c70bd924170041f01b8244789eb383c03bbcb4248fe4a67abaceffe398ec5e0cb93c4e656d82dd5f34f458b9622c@34.12.126.11:30303 \
  --abc.sequencer-http https://rpc.camp.raas.gelato.cloud
```
