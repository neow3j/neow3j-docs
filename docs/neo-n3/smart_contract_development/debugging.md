# Debugging

By default, the `neow3jCompile` Gradle task will output a `.nefdbgnfo` file that contains debugging information for the compiled smart contract. The file is placed in the same folder as the NEF and contract manifest, i.e., usually at `./build/neow3j`. The file is actually a zip archive containing a JSON file, if you want to take a look inside.

The debugging information is meant to be used with the [Neo Debugger](https://github.com/neo-project/neo-debugger), which is part of the VS Code extension pack called [Neo Blockchain Toolkit](https://marketplace.visualstudio.com/items?itemName=ngd-seattle.neo-blockchain-toolkit). Smart Contract debugging is not possible in IntelliJ. From here on we assume you are working in VS Code and have the Neo Blockchain Toolkit extension installed.

For VS Code to know what contract to debug, you need to add a launch configuration. Launch configuration are in the file `.vscode/launch.json` in your projecet directory. If there is no such file yet, you can generate it by selecting "Add Configuration" from the "Run" menu. In `launch.json` move your cursor inside the "configurations" section and use autocompletion, such that an input box will appear containing an entry "*Neo Contract: Launch*". Select it, and a new configuration will be added. Adapt the configuration similarly to the following example. The `program` property has to point to the NEF file that was output by the neow3j Gradle plugin. For more information on how to configure a launch configuration for the Neo Debugger head over to one of the creator's [examples](https://github.com/devhawk/safe-purchase-sample/blob/master/.vscode/launch.json).

Here's the excerpt from neow3j's boilerplate project.

```json
{
    "name": "HelloWorldSmartContract",
    "type": "neo-contract",
    "request": "launch",
    "program": "${workspaceFolder}/build/neow3j/HelloWorld.nef",
    "neo-express": "${workspaceFolder}/default.neo-express",
    "invocation": {
        "operation": "getOwner",
        "args": []
    },
    "storage": [
        {
            "key": "owner",
            "value": "@NNSyinBZAr8HMhjj95MfkKD1PY7YWoDweR"
        }
    ]
}
```

With the configuration in place you can start debugging by pressing F5 or use "Debug: Start Debugging" from the Command Palette.

> **Note**  
Currently, debugging is only supported inside of the contract class. If the class makes calls to methods in the devpack or in other custom classes in your workspace, the debugger will not step into those methods.
