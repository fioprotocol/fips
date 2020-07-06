---
fip: FIP number X
title: FIP Move Action Mapping Into State.
status: FIP status
type: FIP type
author: Ed Rotthoff (ed@dapix.io) 
created: 7/6/2020
updated: 
---

## Abstract
Presently in the FIO protocol, adding a new action to the protocol is a forking change. This is due to the action mapping (which enforces the legal action set) being hardcoded into the FIO core blockchain code.
* Plan the transition to using the new design (which is a mandatory upgrade for all BP and API and users of the FIO protocol).
* Add logic into the protocol to use the new logic after some pre-defined block time.
* Add new endpoint add_action
* Add new endpoint remove_action
* Add new endpoing get_actions
* Publish guidelines for the BPs and the community as to how to roll out new actions in the FIO protocol.


Proposed new actions:
|Action|Endpoint|Description|
|---|---|---|
|add action|add_action|This endpoint will add a new action into the fio protocol.|
|remove action|remove_action| This endpoint will remove the specified action from the fio protocol.|
| get actions |get_actions |This endpoint will list the actions permitted in the FIO protocol, with paging integration.|

## Terminology
* **terms** - define them here
* **more terms** - define them here

## Motivation
This FIO protocol presently uses a list of actions to prohibit unknown actions from spamming the protocol. The list of actions is implemented in the FIO protocol core code (this was decided by the team to get to main net delivery more quickly). Because the list is in the core code, any actions added to the protocol require a mandatory upgarde of the protocol version. This is undesirable because we cannot absolutely control the version enforcement, and it is VERY inconvenient  (even un-feasible) for certain entities such as exchanges to frequently upgrade their code version. It is FAR more desirable to have the changes necessary for new action addition performed by the BP's without any upgrade required of the various entities using the FIO protocol.

* Mandatory and frequent upgrades of node version are un-feasible for the community.
* Updates to the protocol performed by BP's to modify state information used by the protocol is a managable solution.

## Specification

### Core Concept 
Create a new table in FIO core state that holds the list of action mappings used by the protocol. Provide setter and getter endpoints that will allow the read, write, and removal of action mappings. Integrate the new table into the blockchain core to enforce allowable actions (be sure to use rock solid soft fork logic which uses a block time identified well ahead of time). Integrate the use of the new action mapping into the serialize json.

#### Major Concept Actions

##### New action: *addaction*
##### New endpoint: /add_action
##### New fee: no fee, this is an administrative action performed by MSIG, bundle-eligible
##### RAM increase: no RAM bump.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|action_name|Yes|name|This is the action name being added to the FIO protocol contracts.|
|contract_string|Yes|string|The name of the contract that owns this action, EX: "fio.token"|

###### Example
```
{
	"action_name": "newaction",
	"contract_string": "fio.token"
}
```
##### Processing

* require auth fio.system accounts (error if the auth is NOT a fio system account)
* verify action name is not empty.
* verify contract string is not empty.
* check if the action name is already in the action mapping, if it is there then 400 error.
* get the current FIO block time.
* add the new action mapping to the actions table in state.
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|

|Invalid action name|action_name is not valid|400|"action_name"|Value sent in, e.g. "newaction"|"Invalid action namevalue"|
|Invalid contract string|contract_string is not valid|400|"contract_string"|Value sent in, e.g. ""|"Invalid contract string value"|
|Invalid authorization|signing authorization is not valid|500|"invalid signature"||"Invalid signature"|

##### Response

|Parameter|Format|Definition|
|---|---|---|
|name of parameter|Format of parameter|Description of what the parameter means.|
|status|String|OK if successful|
###### Example
```
{
	"status": "OK"
}
```
##### New action: *remaction*
##### New endpoint: /remove_action
##### New fee: no fee, this is an administrative action performed by MSIG, bundle-eligible
##### RAM increase: no RAM bump.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|action_name|Yes|name|This is the action name being added to the FIO protocol contracts.|

###### Example
```
{
"action_name": "newaction"
}
```
##### Processing

* require auth fio.system accounts (error if the auth is NOT a fio system account)
* verify action name is not empty.
* check if the action name exists in the action mapping, if NOT then this is a 400 error.
* remove the action mapping from the actions table in state.
##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|

|Invalid action name|action_name is not valid|400|"action_name"|Value sent in, e.g. "newaction"|"Invalid action namevalue"|
|action not found|action not found |400|"action_name"|Value sent in, e.g. "newaction"|"action not found."|
|Invalid authorization|signing authorization is not valid|500|"invalid signature"||"Invalid signature"|

##### Response

|Parameter|Format|Definition|
|---|---|---|
|name of parameter|Format of parameter|Description of what the parameter means.|
|status|String|OK if successful|
###### Example
```
{
"status": "OK"
}
```

##### New action: *get_actions*
##### New endpoint: /get_actions
##### New fee: no fee, this is a get call in the chain plugin
##### RAM increase: no RAM bump.
##### Request
|Parameter|Required|Format|Definition|
|---|---|---|---|
|limit|no|number|This is the number of rows to display|
|offset|no|number|This is the action id at which to start row display|

###### Example
```
{
"limit": 5,
"offset": 2
}
```
##### Processing

* anyone can call this endpoint.
* verify limit and offset if they are present (not negative, if negative 400 error)
* query the table in state and return the resulting rows.

##### Exception handling
|Error condition|Trigger|Type|fields:name|fields:value|Error message|
|---|---|---|---|---|---|

|Invalid limit|limit is not valid|400|"limit"|Value sent in, e.g. "-1"|"Invalid limit value"|
|invalid offset|offset not valid |400|"offset"|Value sent in, e.g. "-1"|"invalid offset value."|


##### Response

|Parameter|Format|Definition|
|---|---|---|
|name of parameter|Format of parameter|Description of what the parameter means.|
|status|String|OK if successful|
###### Example
```
{
"actions":[
          {
          "action_name": "newaction1",
          "contract_string":"fio.token",
          "blocktimestamp": 1234456667
          },
          {
          "action_name": "newaction2",
          "contract_string":"fio.token",
          "blocktimestamp": 1234456765
          },
          {
          "action_name": "newaction3",
          "contract_string":"fio.token",
          "blocktimestamp": 1234456765
          }
          ],
          more: true
}
```



#### Changes to existing actions
serialize_json -- modify serialize json to use the mapping from state instead of using the action mapping coded into the code. 

## Rationale
We create a new table in state with primary key Id, and secondary key being the action name, this will allow adequate paging and also adequate lookup by action name.

## Implementation
Implementation Details --
Add a new table to the eosio_contract that holds FioAction objects (see below).
— add a new class called fio_action.hpp (use account_object as example).
use fioAction object below.
– add new types to types.hpp
–  add your new index to the controller.cpp indices.

Add a new callback for the action into the eosio_contract.cpp
— add new apply_eosio_updateacts to eosio_contract.cpp
– add new type to contract_types.hpp

Add the new action to the eosio_contract_abi file.
— add new updateacts action to eosio_contract_abi.cpp

Plug in the contract action handler, into the controller.
— add new SET_APP_HANDLER call to controller.cpp (around line 340)

Add the new post action callback to the eosio system contract.
— add new action updateacts to native.hpp for system contract.
this action just needs one line check auth fio system contract.
— add new struct to contract_types.hpp for updateacts params and get_account and get_name

Data structures--
— FioActions contains a vector<FioAction>
— FioAction is a struct 
— name actionname
— string contractnam
— uint64  timestamp.  Block time when added 
(set to zero for “available at main net”)


apply_context.cpp
Apply context —
get the present block time.
get the action from the tx.
if its after the hard fork block, and if there is data in the actions table (note set this block time to be some time after the community is notified, to allow time for the MSIG to update the actions table to execute, the only check necessary might be to just check that the actions table is not empty). This logic could be further distilled to just check for the existence of the new table (with records), and if its there then use it, otherwise do the old logic.
read the actions table by name.
If action doesn’t exist in actions table then error.
If action exists in actions table 
If present block time <= timestamp Error
else if its before the hard fork block time, or if the actions table is empty
do the existing logic.



chain_plugin.cpp
Serialize json — 
if there are entries in the actions table, or if its after a pre-determined block time then 
Use db.get style access to check if the action is in the actions table.
If its not in the list then error.
If its there then get the contract name and use it.
else
Use the existing logic.



## Backwards Compatibility
Ensure that a soft fork plan is put into place that identifies the block time after which the MSIG to set the new contract into place will be executed.  Two drop dead times need to be identified, first, the date/time at which all BPs, API nodes, and exchanges must be updated with the new version, then secondly the date/time by which the MSIG will be executed to put into place the new contract version. This second block time will be the time coded into the forking logic in the code.

## Future considerations
This section may include proposals for how the new functionality could be enhanced in the future.
