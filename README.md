# Polkascan Open-Source

Polkascan Open-Source Application

<hr></hr>

## Launch Polkascan with a spartan-node-template connected to testnet.

Telemetry: https://telemetry.polkadot.io/#map/Spartan%20testnet

### Step 1: Clone repository:

```bash
git clone https://github.com/subspace/polkascan-os.git -branch subspace-testnet
```

### Step 2: Change directory:

```bash
cd polkascan-os
```

### Step 3: Make sure to also clone submodules within the cloned directory:

```bash
git submodule update --init --recursive
```

### Step 4: Then build docker containers

Start mysql

```bash
docker-compose -p subspace-testnet -f docker-compose.subspace-testnet.yml up -d mysql
```

Start services

```bash
docker-compose -p subspace-testnet -f docker-compose.subspace-testnet.yml up --build
```

Give it a time to connect nodes and start fetching blocks.

```
harvester-worker_1       | [2021-08-24 03:34:58,002: WARNING/ForkPoolWorker-8] + Added 0xeafd5844adcb42bc9687fe83f8f2d319d3d331a50a89a415877d1e79257a9454
harvester-worker_1       | [2021-08-24 03:34:58,010: INFO/ForkPoolWorker-8] Task app.tasks.accumulate_block_recursive[23dce8bf-9e70-4eba-9bd4-465256f7996c] succeeded in 0.6836780510202516s: {'result': '10 blocks added', 'lastAddedBlockHash': '0x0fc595326ca904336d6c789781f1ec4d23de7c0243f71b95bfd67e086e8593f2', 'sequencerStartedFrom': False}
```

<hr></hr>

## Launch Polkascan with a local spartan-node-template for development.

### Step 1: Clone repository:

```bash
git clone https://github.com/subspace/polkascan-os.git -branch subspace-testnet
```

### Step 2: Change directory:

```bash
cd polkascan-os
```

### Step 3: Make sure to also clone submodules within the cloned directory:

```bash
git submodule update --init --recursive
```

### Step 4: Runing your local development network

#### 4.1 node-template-spartan.

<hr></hr>

This docker-compose files need to now you localhost node IP, make sure to create an ENV variable.
Note that this IP its the reference from the docker container to you host.

```bash
export LOCAL_SPARTAN_NODE=$(ip addr show | grep "\binet\b.*\bdocker0\b" | awk '{print $2}' | cut -d '/' -f 1)
# out: LOCAL_SPARTAN_NODE=127.0.0.1
```

For the [node-template-spartan](https://github.com/subspace/substrate/tree/poc/bin/node-template-spartan) **poc branch**

```
git clone https://github.com/subspace/substrate.git -b poc
```

Make sure the node has been launched with:

```
cargo +nightly run --bin node-template-spartan -- --dev --tmp --rpc-external --rpc-cors=all --ws-external --rpc-methods=Unsafe --pruning=archive
```

#### 4.2 spartan-farmer.

<hr></hr>

For the [spartan-farmer](https://github.com/subspace/spartan-farmer) **w3f-spartan-ms-3.0 branch**

```
git clone https://github.com/subspace/spartan-farmer.git -b w3f-spartan-ms-3.0
```

Follow the [instructions](https://github.com/subspace/spartan-farmer/blob/w3f-spartan-ms-3.0/README.md) to install run the farmer and start the blocks production.

```
# in case you need it
spartan-farmer erase-plot

# generate a new plot
spartan-farmer plot 256000 spartan

# start the farmer
spartan-farmer farm

```

### Step 5: Then build the docker containers.

Finally run the Polkascan docker-compose and run all the services.

Start mysql

```bash
docker-compose -p subspace-local -f docker-compose.subspace-local.yml up -d mysql
```

Start services

```bash
docker-compose -p subspace-local -f docker-compose.subspace-local.yml up --build
```

## Links to applications

<hr></hr>

- Polkascan Explorer GUI: http://127.0.0.1:8080
- Monitor harvester progress: http://127.0.0.1:8080/kusama/harvester/admin
- Harvester Task Monitor: http://127.0.0.1:5555

## Other networks

<hr></hr>

If you are looking to run another networks check the original Polkascan project:

- https://github.com/polkascan/polkascan-os

## Add custom types for Substrate Node Template

<hr></hr>

- Modify `harvester/app/type_registry/custom_types.json` to match the introduced types of the custom chain
- Truncate `runtime` and `runtime_*` tables on database
- Start harvester
- Check http://127.0.0.1:8080/node-template/runtime-type if all type are now correctly supported
- Monitor http://127.0.0.1:8080/node-template/harvester/admin if blocks are being processed and try to restart by pressing "Process blocks in harvester queue" if process is interupted.

## Cleanup Docker

<hr></hr>

Use the following commands with caution to cleanup your Docker environment.

### Prune images

```bash
docker system prune
```

### Prune images (force)

```bash
docker system prune -a
```

### Prune volumes

```bash
docker volume prune
```

## API specification

<hr></hr>

The Polkascan API implements the https://jsonapi.org/ specification. An overview of available endpoints can be found here: https://github.com/polkascan/polkascan-pre-explorer-api/blob/master/app/main.py#L60

## Troubleshooting

<hr></hr>

When certain block are not being processed or no blocks at all then most likely there is a missing or invalid type definition in the [type registry](https://github.com/polkascan/polkascan-pre-harvester/blob/c5f544ad631e3754ba1e818a26b7aac1ef11f287/app/type_registry/custom_types.json).

Some steps to check:

- The Substrate node should be started in archive mode (`--pruning archive`)
- Be sure to regularly update, also the Git submodules: `git submodule update --init --recursive` for latest version of py-scale-codec
- Check which blocks are not being processed and try to restart manually at: http://127.0.0.1:8080/<NETWORK_NAME>/harvester/admin
- Check Celery queue for more details about failing blocks: http://127.0.0.1:5555/tasks?state=FAILURE
- Check and match types in [./harvester/app/type_registry/custom_types.json](https://github.com/polkascan/polkascan-pre-harvester/blob/c5f544ad631e3754ba1e818a26b7aac1ef11f287/app/type_registry/custom_types.json) with added types in Substrate

You can also dive into Python to pinpoint which types are failing to decode:

```python
import json
from scalecodec.type_registry import load_type_registry_file
from substrateinterface import SubstrateInterface

substrate = SubstrateInterface(
    url='ws://127.0.0.1:9944',
    type_registry_preset='substrate-node-template',
    type_registry=load_type_registry_file('harvester/app/type_registry/custom_types.json'),
)

block_hash = substrate.get_block_hash(block_id=3899710)

extrinsics = substrate.get_block_extrinsics(block_hash=block_hash)

print('Extrinsincs:', json.dumps([e.value for e in extrinsics], indent=4))

events = substrate.get_events(block_hash)

print("Events:", json.dumps([e.value for e in events], indent=4))
```
