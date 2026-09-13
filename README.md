W017911 Godrej Traceability - Node-RED

Overview

This repository contains the Node-RED application used for the Godrej traceability system.

The Node-RED flows collect process data from plant equipment through OPC UA and MQTT, process the data, and write traceability / machine data into MySQL databases.

The current flow configuration contains multiple process areas including:

Mixer PLC

Cathode Mixer

Powder Handling

Liquid Handling

Anode Oven

Cathode Oven

PHS Anode

PLC 1, PLC 2 and PLC 3

Winding PLC

EMS

Machine-data flows for the above process areas

Battery/cell traceability and mapping logic

Home-screen data insertion

The exported flow configuration also contains disabled/test flows.

## **Repository Structure**
```
W017911_godrej_traceability_nodered/
│
├── lib/                         # Project/application supporting code
├── node_modules/                # Installed dependencies - NOT versioned
│
├── .config.nodes.json           # Node-RED project configuration/flow data
├── .config.runtime.json         # Node-RED runtime/flow data
├── .config.users.json           # Node-RED users/configuration data
│
├── *.backup                     # Automatically generated backups - NOT versioned
│
├── package.json                 # Node.js project dependencies
├── package-lock.json            # Locked dependency versions
├── settings.js                  # Node-RED runtime configuration
└── README.md
```
The exact flow file used by Node-RED is determined by settings.js. Do not rename the .config.* files unless the corresponding settings.js configuration is changed.

### Main Integrations

#### OPC UA

The flows communicate with multiple PLC/machine OPC UA servers.

The current configuration includes OPC UA endpoints for equipment such as:
```
-- Anode Mixer

-- Cathode Mixer

-- Anode Thickness

-- Cathode Thickness

-- Anode Oven

-- Cathode Oven

-- PHS Anode

-- PHS Cathode

-- Powder Handling

-- Liquid Handling

-- Laponite

-- PVA Lamination

-- PLC 1

-- PLC 2

-- PLC 3

-- Winding PLC
```
OPC UA security in the exported configuration is currently configured as None for the listed endpoints.

## MQTT

The EMS-related flows use an MQTT broker.

Current configuration:
```
Broker: mqtt://10.18.16.209
Port:   1883
```
**Example MQTT topic used by the flows:**
```
mfm/LT_PANEL
```

If the broker IP changes on a new machine/site, update the MQTT broker configuration inside Node-RED.

## MySQL

The flows use MySQL databases on the local machine.

Databases referenced by the current flow configuration include:

godrej_traceability
godrej_ems
godrej_machinedata

The current Node-RED configuration uses:
```
Host: 127.0.0.1
Port: 3306
```
The MySQL databases and their tables are external dependencies of this repository. A database backup/schema should be maintained separately from the Node-RED Git repository.

### Node-RED Dependencies

The exported flow configuration declares these Node-RED modules:
```
node-red-node-mysql       3.0.0
node-red-contrib-opcua    0.2.348
node-red-node-ping        0.3.3
```
The preferred installation method is to use the repository's package.json and package-lock.json rather than installing arbitrary/latest versions manually.

After cloning the repository:

npm ci

If these modules are not present in package.json, install them explicitly:

npm install node-red-node-mysql@3.0.0
npm install node-red-contrib-opcua@0.2.348
npm install node-red-node-ping@0.3.3

# **Running on a New Machine -->**


The following procedure assumes a completely fresh Windows machine.

1. Install prerequisites

Install:
```
Git

Node.js LTS

MySQL Server

Any required MQTT broker/network infrastructure

Node-RED dependencies through npm
```

Verify Node.js and npm:

node --version
npm --version

Verify Git:

git --version

2. Clone the repository

Open PowerShell or Command Prompt:
``` bash
git clone <REPOSITORY_URL>
```

Example:
``` bash
git clone https://<git-server>/<repository>.git
``` 
Enter the project directory:
``` bash
cd W017911_godrej_traceability_nodered
```
3. Install Node.js dependencies

Run:
```bash
npm ci
```
Do NOT copy node_modules from the old machine.

npm ci recreates node_modules from package-lock.json.

If npm ci fails because the lock file and package.json are inconsistent, use:

npm install

Then commit any intentional dependency/lock-file changes.

4. Configure Node-RED

The repository contains:

settings.js

Review this file before starting Node-RED.

In particular, verify:

Node-RED user directory

Flow file name

HTTP port

Authentication configuration

Context storage

Any local file paths

Environment-specific settings

The flow file configured in settings.js must exist in the project directory.

5. Restore / prepare MySQL

Install MySQL Server and make sure it is running on:
```sql
127.0.0.1:3306

Create the required databases:

CREATE DATABASE godrej_traceability;
CREATE DATABASE godrej_ems;
CREATE DATABASE godrej_machinedata;
```
Then restore the required table structure and data from the project's database backup.

The Node-RED flow references tables such as:

winding_plc
cell_main
battery_cell_mapping
plc_3
plc_status
batch_main
electrode

The complete database schema is not contained in the Node-RED flow export, so the database backup/schema must be restored separately.

6. Configure MySQL credentials

Open the MySQL configuration nodes in Node-RED and configure the username/password required by the local MySQL installation.

Do not commit passwords or other secrets to Git.

If the existing Node-RED credential mechanism is used, restore the required credential file/configuration separately.

7. Configure network connectivity

Before starting production flows, verify that the new machine can reach:

All required PLC OPC UA endpoints

MQTT broker

MySQL server

Any other external services used by the flows

For example, verify that the MQTT broker is reachable:

10.18.16.209:1883

The OPC UA endpoint IP addresses and ports are stored in the Node-RED flow configuration. If the machine/PLC network is different, update the corresponding OPC UA endpoint nodes.

8. Start Node-RED

From the project directory:

node-red -u .

If the project contains a locally installed Node-RED executable, you can also use:

npx node-red -u .

Node-RED should start using the project directory as its user directory.

Open the Node-RED editor in a browser using the HTTP address/port configured in settings.js.

Example:

http://localhost:1880

Use the actual configured port if different.

9. Verify the flows

After starting Node-RED:

Open the Node-RED editor.

Confirm all required tabs are present.

Confirm required flows are enabled.

Confirm OPC UA nodes connect successfully.

Confirm MQTT input nodes receive messages.

Confirm MySQL nodes connect successfully.

Check the Node-RED debug/status messages.

Verify that test/manual flows are not accidentally enabled in production.

Verify that traceability records are being written correctly.

Production Startup

For a production machine, Node-RED should normally run as a Windows service rather than requiring a user to start it manually.

If using NSSM:

Application:
node.exe

Arguments:
<path-to-node-red>\node_modules\node-red\red.js -u <project-directory>

Startup directory:
<project-directory>

Alternatively, if Node-RED is installed globally, configure the appropriate Node-RED executable and arguments for the installation.

After configuring the service:

Start service
→ Check Node-RED logs
→ Open Node-RED editor
→ Verify OPC UA/MQTT/MySQL connections

Git Version Control

The purpose of this repository is to keep the Node-RED application and flows versioned.

Files that should normally be committed

.config.nodes.json
.config.runtime.json
.config.users.json       # only if it does not contain secrets
package.json
package-lock.json
settings.js
lib/
README.md

The exact files to commit depend on which .config.* file is configured as the actual flow file in settings.js.

Files that should NOT be committed

node_modules/
*.backup
*.log
.env
.env.*

Credentials and passwords should also not be committed.

Check Git status

git status

Save a flow change

After making changes in Node-RED:

git status
git add .
git commit -m "Update Node-RED flows"
git push

Use meaningful commit messages, for example:

Update PLC-3 machine data flow
Fix winding PLC traceability insertion
Update OPC UA endpoint configuration
Fix battery cell mapping
Add EMS data processing

Updating an Existing Machine

To deploy a newer version to an already configured machine:

git pull
npm ci

Then restart the Node-RED service.

After restart, verify:

Node-RED starts without errors

Flow tabs are present

OPC UA connections are healthy

MQTT connection is healthy

MySQL connection is healthy

Traceability data is being recorded

Rollback

Git can be used to restore an earlier known-good version.

View history:

git log --oneline

Temporarily inspect an older version:

git checkout <COMMIT_ID>

For production rollback, stop Node-RED before replacing the active flow/configuration and restart it after verification.

A safer long-term approach is to create release tags for known-good production versions:

git tag v1.0.0
git push origin v1.0.0

Backup Strategy

This repository provides versioned backup of the Node-RED application and flows.

Git stores changes such as:

Version 1
   ↓
PLC flow added
   ↓
Version 2
   ↓
Database query modified
   ↓
Version 3
   ↓
OPC UA endpoint updated

Do not rely on the automatically generated *.backup files as the primary backup mechanism.

The recommended backup layers are:

Git repository - Node-RED source/configuration/flows

Database backup - MySQL schema and data

Production configuration backup - machine-specific settings/secrets

Release tags - known-good production versions

Troubleshooting

Node-RED does not start

Run Node-RED manually from the project directory:

node-red -u .

Read the console error before starting the Windows service again.

Missing Node-RED nodes

If the editor reports unknown node types:

npm ci

Then verify that the required modules are installed:

npm list node-red-node-mysql
npm list node-red-contrib-opcua
npm list node-red-node-ping

MySQL connection failure

Verify:

MySQL service is running
Host: 127.0.0.1
Port: 3306
Database exists
Username/password are correct

OPC UA connection failure

Verify:

PLC is powered and reachable
IP address is correct
OPC UA server is running
Port is correct
Network/firewall allows the connection

MQTT connection failure

Verify:

Broker is running
Broker IP is correct
Port 1883 is reachable
Required MQTT topics are publishing

Important Production Notes

Do not modify production flows directly without first creating a Git commit/version.

Before major changes, create a known-good Git tag.

Do not commit passwords, API keys, certificates/private keys, or other secrets.

Do not commit node_modules.

Keep MySQL backups separate from the Node-RED repository.

Test flow changes before enabling them in production.

Be especially careful when changing OPC UA endpoints, SQL queries, trigger logic, or database table names.

Current Flow Dependencies

The supplied flow export declares the following Node-RED modules:

node-red-node-mysql       3.0.0
node-red-contrib-opcua    0.2.348
node-red-node-ping        0.3.3

The flow contains OPC UA, MQTT, MySQL, function, switch, inject, debug and related Node-RED nodes.

First-Time Installation Checklist

Use this checklist when setting up a completely new machine:

[ ] Install Windows prerequisites
[ ] Install Git
[ ] Install Node.js LTS
[ ] Install/configure MySQL
[ ] Clone Git repository
[ ] cd into project directory
[ ] Run npm ci
[ ] Verify settings.js
[ ] Verify configured flow file exists
[ ] Restore MySQL databases/schema
[ ] Configure MySQL credentials
[ ] Verify OPC UA network connectivity
[ ] Verify MQTT connectivity
[ ] Start Node-RED manually
[ ] Verify all required flows
[ ] Test database insertion
[ ] Test OPC UA communication
[ ] Test MQTT communication
[ ] Configure NSSM/Windows service if required
[ ] Start production service
[ ] Verify application after restart
[ ] Create a Git tag for the known-good deployment

Repository Maintenance

When adding or changing Node-RED functionality:

# Check current changes
git status

# Review changes
git diff

# Stage
git add .

# Commit
git commit -m "Describe the change"

# Push
git push

Always review git status before committing to ensure that generated backups, node_modules, credentials, and other unwanted files are not being committed.