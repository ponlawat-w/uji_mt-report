[« Back](../README.md)

# Experiment 3: Scenario

## Contents

- [Running Fog Ethereum Node in PC](#ex-eth-pc)
- [Deploying Smart Contracts](#ex-smart-contracts)
- [Running Fog API in PC](#ex-api-pc)
- [Running Fog Ethereum Node in Raspberry Pi](#ex-eth-pi)
- [Initialising Edge Devices](#ex-init-edge)
- [Experiment Instructions](#ex-exp-ins)

---

## Running Fog Ethereum Node in PC
<a id="ex-eth-pc"></a>

*※ This instructions contain PowerShell script in window. For executing them in other platform, see in each `.ps1` powershell script file and follow the command inside.*

1. Clone repository [https://github.com/ponlawat-w/uji_mt-contracts](https://github.com/ponlawat-w/uji_mt-contracts)

```
git clone https://github.com/ponlawat-w/uji_mt-contracts.git
```

2. Go to `network` directory
```
cd ./network
```

3. Run `script-init.ps1` to create a new node, give node number `0`, `1`, `2`, or any.
```
./script-init.ps1
> Node Number: 1
> Password: **Ethereum Private Key Passphrase Here**
```

4. Run `script-init.ps1` again for another node, or run the script from another device for testing across the network.

5. Directory `./node[NODE_NUMBER]` will be created. For example `./node1` or `./node2`.

6. Run `script-run-local.ps1`

```
./script-run-local.ps1
> Node Number: 1
> Bootnode (if any): 
> Auto mining? (y or leave blank): y
```

- **Bootnode**: leave blank because it is the first node to be run
- **Auto mining**: enter `y` if want the current node to be a miner, or leave blank if want it to be just a node

7. In the output stream of geth, copy, or take note the enode address of P2P networking. In the example below, the enode address to be saved is `enode://xxxyyyzzz@127.0.0.1:30303`

```
INFO Started P2P networking  self=enode://xxxyyyzzz@127.0.0.1:30303
```

8. (optional) If there is other Ethereum node run on PC, repeat step 6, but in the `Bootnode` input, enter the enode address derived from step 7. If the node is run on different device, replace `127.0.0.1` to the IP address of the first node.

---

## Deploying Smart Contracts
<a id="ex-smart-contracts"></a>

1. Make sure that geth client is running.

2. Go to root directory of `contracts` repository ([https://github.com/ponlawat-w/uji_mt-contracts](https://github.com/ponlawat-w/uji_mt-contracts)).

3. Update file `truffle-config.js`, update settings in `localNode` to be connection data of the geth client which is running, or add a new setting and name it differently.

```json
localNode: {
  host: '127.0.0.1',
  port: 8545,
  network_id: 2564,
  password: true
}
```

4. Copy file `passwords-dist.js` to `passwords.js` (which is gitignored).

5. In `passwords.js`, enter passphase that used to encrypt private key in step 3 of section [initialising Ethereum in PC](#ext-eth-pc). The password has to be entered correspondently to the network name set in `truffle-config.js`.

```js
module.export = {
  localNode: 'passwordHere',
  localNode1: '',
  localNode2: ''
};
```

6. In `./migrations/1_deploy_all.js` line `23` to `38`, make sure that `FOR TESTING` is commented and `FOR DEPLOYMENT` is uncommented.
```js
// ---------------------------------------------
// FOR TESTING (uncomment for `truffle test`)
// const S2Regions = artifacts.require('S2Regions');
// const GeohashRegions = artifacts.require('GeohashRegions');
// const Regions = S2Regions;
// await deployer.link(Utils, [S2Regions, GeohashRegions, Devices]);
// await deployer.deploy(S2Regions);
// await deployer.deploy(GeohashRegions);
// await deployer.link(Regions, Devices);
// ---------------------------------------------
// FOR DEPLOYMENT (uncomment for `truffle deploy/migrate --f 1`)
const Regions = artifacts.require(require('../regions-artifact')(networkName));
await deployer.link(Utils, [Regions, Devices]);
await deployer.deploy(Regions);
await deployer.link(Regions, Devices);
// ---------------------------------------------
```

7. In `./regions-artifact.js` line `7`, update `defaultRegions` value to be either `GeohashRegions` or `S2Regions`. (In the experiment 3 of master thesis report, the S2 one was used)
```js
const defaultRegions = 'S2Regions'; // or 'GeohashRegions'
```

8. Make sure that ethereum account created has some ether money enough to deploy the contracts. If not, mine some blocks to get money first.

Firstly, enter geth console:
```
geth attach http://127.0.0.1:8545
```

In geth console, start mining (`4` is thread number, change it up to PC hardware specifications):
```
> miner.start(4);
```

Look at geth console output, if there are messages like "new block is mined", the miner can now be stopped.
```
> miner.stop();
```

9. After making sure that the account has enough money to deploy contracts, on the root directory of `contracts` repository, enter truffle console on the configured network:
```
truffle console --network localNode
```

10. Deploy the contracts from migration number 1
```
deploy --f 1
```

11. Wait until all Smart Contracts are deployed. If there is no progress after having waited for a long time, it is possible that the geth client miner is not running. Open a new terminal and repeat step 8 to mine a new block that propagates the Smart Contracts deployment.

---

## Running Fog API in PC
<a id="ex-api-pc"></a>

1. In root directory of repository `contracts` ([https://github.com/ponlawat-w/uji_mt-contracts](https://github.com/ponlawat-w/uji_mt-contracts)), enter truffle console using the network that was used to deploy the Smart Contracts in the previous section.
```
truffle console --network localNode
```

2. Execute `export-api-config.js` to create API config file.
```
exec ./export-api-config.js
```

3. The result (API config) will be located at `./out/api-config.json`.

4. Leave the repository directory, clone `fog-api` repository from [https://github.com/ponlawat-w/uji_mt-fog_api](https://github.com/ponlawat-w/uji_mt-fog_api).

5. Copy `api-config.json` from `./out` directory in `contracts` repository obtained from step 2 - 3, to the root directory of `fog-api` repository.

```
git clone https://github.com/ponlawat-w/uji_mt-fog_api.git
```

6. From root directory of `fog-api` repository, run node to serve Fog API:
```
npm start
```
or
```
node index.js
```

7. Test API by entering `http://127.0.0.1/alive`, it should return a json object with status `true`.

8. Test API connection to ethereum by entering `http://127.0.0.1/balance/get/ETHADDR?unit=ether`, replacing `ETHADDR` to be Ethereum address (starts with `0x`). The API should return a json object with value to be money balance in the account.

---

## Running Fog Ethereum Node in Raspberry Pi
<a id="ex-eth-pi"></a>

1. Clone repository [https://github.com/ponlawat-w/uji_mt-raspi_scripts](https://github.com/ponlawat-w/uji_mt-raspi_scripts)

```
git clone https://github.com/ponlawat-w/uji_mt-raspi_scripts.git
```

2. If necessary, change mode .sh files by mark them to be executable:
```
chmod u+x ./*.sh
```

3. Install dependency packages:
```
sudo ./0-install-packages.sh
```

4. Install geth client:
```
sudo ./1-setup-geth.sh
```

5. Copy text file `config-dist` and rename the new one to `config`, place it in the same direcory.

6. Make sure that there is an API running in other device (e.g. on PC from the previous section), and it is accessible by an IP address.

7. In `config` file, update `APIURL` to be API URL base of the running API. For example, if the API is running on the device with IP `192.168.1.120`, update the file to following:
```
# Base URL of running API
APIURL="http://192.168.1.120:80"
```

8. Initialise the Ethereum network:
```
./2-setup-blockchain.sh
```

9. Create a new Ethereum address:
```
./3-blockchain-new-account.sh
```

10. From the previous step, a new Ethereum public address (starts with `0x`) will be generated and printed on the console. Put the address to `config` file at `ETHADDR`:
```
# Ethereum address obtained from step 3
ETHADDR="0xabcdefg..."
```

11. Set up API:
```
sudo ./4-setup-api.sh
```

12. Register geth and API as services:
```
sudo ./5-register-services.sh
```

13. Geth client in Raspberry PI needs to know enode address of a running node in the Ethereum network for establishing a peer-to-peer connection. The address can be set manually through geth console, or can simpler use the script by the following steps:

Firstly, **in PC *(not Raspberry PI)***, go to `network` directory in `contracts` repository.

Then, run PowerShell script `script-forward-settings.ps1`:
```
./script-forward-settings.ps1
```

Select a network interface whose IP address will be discovered by Raspberry PI, by entering the number when prompt:
```
[12]: Wi-Fi 2 - 172.0.31.128
[16]: Wi-Fi - 169.254.192.255
[49]: vEthernet (Default Switch) - 172.28.64.1
[69]: vEthernet (WSL) - 172.21.144.1
Select current network interface: 12
```

Leave auto mining or enter `false`, because Raspberry PI cannot mine a block:
```
Auto Mining? (true or false, blank for false): false
```

Enter SSH connection (`user@ip`) of the target Raspberry PI (for example if Raspberry PI is running on IP address `172.0.31.64`):
```
Target SSH (e.g. pi@xx.xx.xx.xx): pi@172.0.31.64
```

Enter Raspberry PI SSH password if prompt.

This script will automatically get enode address from the current device, and send it to the target Raspberry PI and restart its geth service to make it discover the Ethereum network.

14. Make sure that geth and API service is running:
```
systemctl status geth.service
systemctl status api.service
```

15. Make sure that API is working by performing step 7 - 8 from [the previous section](ex-api-pc).

---

## Initialising Edge Devices
<a id="ex-init-edge"></a>

1. Clone repository [https://github.com/ponlawat-w/uji_mt-edge_devices](https://github.com/ponlawat-w/uji_mt-edge_devices):
```
git clone https://github.com/ponlawat-w/uji_mt-edge_devices.git
```

2. Go to `service-provider` directory.

3. Enter file `src/service-provider.ino`.

4. In the service provider board, make sure that `A0` pin is unconnected, because the unstability from an unconnected pin is used to generate a random value to sign the transaction. If `A0` pin cannot be used to serve this purpose, change the value of the defined variable `UNCONECCTED_RANDOM_PIN`.

5. Make sure that temperature (DHT) sensor is connected to `D2`. If it is not possible, update the defined variable `SENSOR_DHT_PIN` to the pin that the sensor is connected.

6. Compile and flash the programme to the device.

7. After flashed, the device should publish `LOCAL_IP` to the Particle cloud with a value to be the local IP address of the device. Note the address as it will be used to communicate with it by edge API.

8. Go to `service-consumer` directory.

9. Enter file `src/service-consumer.ino`.

10. In the service consumer board, make sure that chainable LED is connected to `D2` and `D3`. If the pins are not available, update the defined variables `LED_PIN1` and `LED_PIN2` to the connected pin.

11. Update variable `FogAPIBaseHostname` to be the IP Address of a fog node.

12. Update array `Locations` to be cell IDs in hexdecimal that are used to simulate the query of this service consumer.

13. Compile and flash the programme to the device. Preferably to power off the device after flashed. The service consumer should start working after all data have been set in the next section.

---

## Experiment Instructions
<a id="ex-exp-ins"></a>

Import [`postman_collection.json`](./postman_collection.json) to Postman to make API calling easier.

### 1. Add Regions Data

1. Call API `FOG_IP/region/register` to register a new region. Make sure to put the correct password in the request header `UnlockPassword`.

2. Use API `FOG_IP/region/add-tree` or `FOG_IP/region/add-cells` to add geocoded cells to the region. In case of using `add-tree`, the tree data can be generated from an array of cell IDs by using API `FOG_IP/utils/make-trees`.

3. Added regions can be tested by API `FOG_IP/regions/list`, `FOG_IP/region/get/[ID]` and/or `FOG_IP/region/query/[cellID]`.

### 2. Register Devices

1. Call API `EDGE_IP/set-eth` to set a Ethereum account and its private key to the service provider device.

2. Call API `EDGE_IP/set-location` to set the service provider device location.

3. Call API `EDGE_IP/set-fog` to set the IP address of a fog node.

4. Call API `EDGE_IP/alive`, the `value` returned should be `3`, which indicates "idle" state of the device.

### 3. Consume Service

1. Call API `EDGE_IP/start-service` to establish a new service consumption. Note the `token` returned from the API.

2. Call API `EDGE_IP/get-service/[TOKEN]` to consume the service (get sensory data). Monitor Fog API console, it should also prints the sensory data observed by the device.

3. Wait until the service get expired, or call API `EDGE_IP/stop-service` to stop the service. In the console monitor of the Fog API should print a random service quality value.

4. Optionally, call fog API `FOG_IP/service/feedback` to give a feedback of the service.

5. Call fog API `FOG_IP/reputation/submit`. The fog node will then update the reputation value by average betweeen random value from step 3 and feedback from step 4 if given, and push the data into the Smart Contract.

6. If the block is mined, the reputation value can be checked by using following APIs:
- `FOG_IP/reputation/get/GET_TYPE/EDGE_ETH_ADDR/SERVICE_NAME/at/CELL_ID`
- `FOG_IP/reputation/get/GET_TYPE/EDGE_ETH_ADDR/SERVICE_NAME/in/REGION_ID`
- where
  - `GET_TYPE` to be `value` or `records`
    - `value` returns the latest reputation value
    - `records` returns all the reputation records
  - `EDGE_ETH_ADDR` to be the Ethereum addres of service provider device
  - `SERVICE_NAME` to be the service name, which is `TEMP` in the thesis
  - `CELL_ID` to be hexidecimal (starts with `0x`) cell ID which is located in the region where the reputation value is required
  - `REGION_ID` to be the ID of the region where the reputation value is required

7. Turn on the service consumer device, observe the LED colour, it should be able to consume when it is requiring from a location where exists the reputation value above the defined threshold.

---
