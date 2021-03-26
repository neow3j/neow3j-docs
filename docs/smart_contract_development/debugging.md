# Debugging

By default, the `neow3jCompile` Gradle task will output a `.nefdbgnfo` file that contains debugging information for the
compiled smart contract. The file is placed in the same folder as the NEF and contract manifest, i.e., usually at
`./build/neow3j`. The file is actually a zip archive containing a JSON file, if you want to take a look inside.

The produced debugging information is meant to be used with the [Neo
Debugger](https://github.com/neo-project/neo-debugger), which is part of the Visual Studio Code (VSCode) extension pack
called [Neo Blockchain Toolkit](https://marketplace.visualstudio.com/items?itemName=ngd-seattle.neo-blockchain-toolkit).
We recommend using VSCode with the Neo Blockchain Toolkit to write, manually test and debug Java smart contracts. 

Once the debugger is installed and a Gradle project for your contract is in place, you must add a
`launch.config` that tells VSCode how to run/debug the contract. To achieve this, select "Add
Configuration" from the "Run" menu. This will create a new launch.json file in the .vscode directory
of your workspace. In that file and inside of the "configurations" array use the shortcut for
autocompletion. An input box will appear that contains an entry "Neo Contract: Launch". Select it,
and a new configuration will be added. Adapt the configuration similarly to the following example.
The `program` property has to point to the NEF file that was output by the neow3j Gradle plugin.
For more information on how to configure a launch configuration for the Neo Debugger head over to
the [Launch Configuration Reference](https://github.com/neo-project/neo-debugger/blob/master/docs/debug-config-reference.md).

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "SmartContract",
            "type": "neo-contract",
            "request": "launch",
            "program": "${workspaceFolder}/build/neow3j/SmartContract.nef",
            "operation": "main",
            "args": [],
            "storage": [],
            "runtime": {
                "witnesses": {
                    "check-result": true
                }
            }
        }
    ]
}
```

With the configuration in place you can start debugging by pressing F5 or use "Debug: Start
Debugging" from the Command Palette.

> **Note**  
> Currently, debugging is only supported inside of the contract class. If the class makes
> calls to methods in the devpack or in other custom classes in your workspace, the debugger will
> not step into those methods.