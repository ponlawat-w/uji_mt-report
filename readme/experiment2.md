[« Back](../README.md)

# Experiment 2: Simulation

This experiment is scripted in contracts repository: [https://github.com/ponlawat-w/uji_mt-contracts](https://github.com/ponlawat-w/uji_mt-contracts).

Clone the repository by following command:
```
git clone https://github.com/ponlawat-w/uji_mt-contracts.git
```

---

## 1. Contracts Deployment

This measures gas used in deploying contracts.

1. Go to root directory of the repository.
2. Enter truffle development console:
```
truffle develop
```

3. Apply migration number 0 *(migration 0 is for this experiment which deploys both Geohash and S2, while migration 1 is for deploying only either Geohash or S2)*
```
deploy --reset --f 0 --to 0
```
  - `--reset` - Replace the current contracts
  - `--f 0` - From migration number 0
  - `--to 0` - To migration number 0

4. The raw results are located at `out/experiment-contracts-geohash.csv` and `out/experiment-contracts-s2.csv`. These files can be used to make a report.

---

## 2. Contracts Simulation

### 2.1. Preparing Regions Data

This section show intructions to generate Geohash and S2 cells from test region polygon data.
The generated data are also pushed to GitHub repository so it is not necessary to perform this step, unless another region polygon dataset is used.

- The polygon data file is located at `./data/regions.geojson` in GeoJSON format.

#### 2.1.1. Geohash

1. Update line `10` in `./data/geohash.js` to the desired Geohash precision:
```js
const PRECISION = 8; // ← Change here
```
2. *From repository root directory*, execute the following command to create Geohash cells from the GeoJSON:
```
node ./data/geohash.js
```
3. The output will be located at directory `./data/out-geohash-PRECISION`, where `PRECISION` is the number of precision defined in step 1.

#### 2.1.2. S2

1. Update line `14` in `./data/s2covering.go` to the desired S2 precision:
```golang
var precision = 19
```

2. *From directory `./data`*, execute the following GoLang script to create S2 cells from GeoJSON:
```
go run s2covering
```

3. *From repository root directory*, execute the following JavaScript to compress the S2 cells:
```
node ./data/s2base64tree.j
```

### 2.2. Generating Random data

The data are generated only once and used for all experiments in 2.3.

1. Go to repository root directory.
2. Execute following JavaScript file to generate random data for the experiments:
```
node ./data/experiment.js
```
3. The generated data file is located at `./data/out/experiment-data.json`

### 2.3. Experiment

This experiment is repeated for the different geocoding techniques (Geohash, S2), and for the different precision. All the experiments use the same dataset generated in 2.2.

1. In `./migrations/1_deploy_all.js` line `23` to `38`, make sure that `FOR TESTING` is commented and `FOR DEPLOYMENT` is uncommented.
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

2. In `./regions-artifact.js` line `7`, update `defaultRegions` value to be either `GeohashRegions` or `S2Regions`.
```js
const defaultRegions = 'GeohashRegions'; // or 'S2Regions'
```

3. In `./experiments.js` line `6` to `8`, update the values of following variables to the desired experiment:
```js
const REGIONS_ARTIFACT = 'GeohashRegions'; // GeohashRegions|S2Regions
const PRECISION_INDEX = 0; // 0|1|2
const ADD_MODE = 'cells'; // cells|tree
```

4. From repository root directory, enter truffle console mode:
```
truffle develop
```

5. Deploy the contracts from migration number 1:
```
deploy --reset --f 1
```

6. In truffle console, execute the experiment, wait until finished:
```
exec ./experiments.js
```

7. The raw results of the experiment will be located in directory `./out`. The files can be used for creating a report.

---

## 3. Mining Test

- Follow log file `./network/experiments-mining/commands.log` to reproduce this experiment.

---

## Test Contracts (Unit Test)

This gives the instructions of running Unit Tests in the Smart Contracts.
The Unit Tests are not included in master thesis report. This is just to test that the Smart Contract functionalities work as expected.

The tests require region data from section 2.1.

1. In `./migrations/1_deploy_all.js` line `23` to `38`, make sure that `FOR TESTING` is uncommented and `FOR DEPLOYMENT` is commented.
```js
// ---------------------------------------------
// FOR TESTING (uncomment for `truffle test`)
const S2Regions = artifacts.require('S2Regions');
const GeohashRegions = artifacts.require('GeohashRegions');
const Regions = S2Regions;
await deployer.link(Utils, [S2Regions, GeohashRegions, Devices]);
await deployer.deploy(S2Regions);
await deployer.deploy(GeohashRegions);
await deployer.link(Regions, Devices);
// ---------------------------------------------
// FOR DEPLOYMENT (uncomment for `truffle deploy/migrate --f 1`)
// const Regions = artifacts.r·equire(require('../regions-artifact')(networkName));
// await deployer.link(Utils, [Regions, Devices]);
// await deployer.deploy(Regions);
// await deployer.link(Regions, Devices);
// ---------------------------------------------
```

2. Enter truffle console:
```
truffle develop
```

3. Execute the following command to test the contract:
```
test
```

---
