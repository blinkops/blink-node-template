# Node Template Plugin

## Running the plugin
In order to run the plugin you wil have to:
```
    go mod verify
    go run main.go
```

## Adding actions
In order to add a new action:
Go to the actions folder and add a new action yaml file, for example this yaml file simulates
an action that takes two numbers as input params and returns the sum of those two numbers:
``` yaml
# sum.yaml
name: "sum"
description: "sum two numbers"
enabled: true
entry_point: "run_node.sh"
parameters:
  first:
    type: "int"
    description: "first number"
    required: true
  second:
    type: "int"
    description: "second number"
    required: true
```

Next, place the script you want to expose by this action (see the entry_point value above):
In this case we are going to call a node script (app.js)
```shell
# run_node.sh
#!/bin/sh
node app.js

```
And this is the node script (app.js - also under the actions folder)
```javascript
var first = process.env.INPUT_FIRST;
var second = process.env.INPUT_SECOND;
console.log("Sum: "+ (parseInt(first) + parseInt(second)));
```

## Build the plugin
You can build a docker version of the plugin in the next way:
```shell
 docker build -f build/Dockerfile . -t "blink-node-plugin:1.0.1"  
```

## Run the plugin
You can run now the plugin via the next command:
```shell
docker run -dp 1337:1337 blink-node-plugin:1.0.1
```
This plugin will listen for incoming commands on port 1337

## Test the plugin
To test the plugin you will need the blink-cli or you can use grpcox (https://github.com/gusaul/grpcox)
which can invoke gRPC calls to your plugins.

Assuming you have the blink plugin-cli (https://github.com/blinkops/blink-cli) you can introspect the plugin actions:
```shell
./plugin-cli ga | jq .
{
  "actions": [
    {
      "name": "sum",
      "description": "sum two numbers",
      "active": true,
      "parameters": [
        {
          "field": {
            "name": "first",
            "type": "int",
            "required": true,
            "description": "first number"
          }
        },
        {
          "field": {
            "name": "second",
            "type": "int",
            "required": true,
            "description": "second number"
          }
        }
      ]
    }
  ]
}
```

You can also test your action by sending commands:
```shell
./plugin-cli e --name sum --parameters '{ "first" : "10", "second" : "20" }'
Sum is: 30
```