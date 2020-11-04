# Debugging

By default the neow3j Gradle plugin will output a `.nefdbgnfo` file that contains debugging
information for the compiled smart contract. It is placed in the same folder as the NEF and contract
manifest, i.e., usually at `./build/neow3j`. The file is actually a zip archive containing a JSON
file, if you want to take a look inside.
The produced debugging information is meant to be used with the 
[Neo Debugger](https://github.com/neo-project/neo-debugger) (on the VSCode marketplace), 
which is part of the Visual Studio Code (VSCode) extension pack called 
[Neo Blockchain Toolkit](https://marketplace.visualstudio.com/items?itemName=ngd-seattle.neo-blockchain-toolkit).
We recommend using VSCode with the Neo Blockchain Toolkit to write Java smart contracts. But the 
debugger can also be installed separately from
[here](https://marketplace.visualstudio.com/items?itemName=ngd-seattle.neo-contract-debug). 

> In order to use the Neo Debugger with the current version of the neow3j compiler you need to
> install a preview version of the debugger, which is not yet on the VSCode marketplace. The release
> can be found on [Github](https://github.com/neo-project/neo-debugger/releases/tag/1.2.28-preview).
> Download the .vsix from that release and follow the official 
> [instructions](https://code.visualstudio.com/docs/editor/extension-gallery#_install-from-a-vsix) 
> to manually install a VSCode extension. In the future these manual steps will 

Once the debugger is installed and a Gradle project for your contract is in place, you must add a
`launch.config` that tells VSCode how to run/debug the contract. To achieve this, select "Add
Configuration" from the "Run" menu. This will create a new launch.json file in the .vscode diractory
of your workspace. In that file and inside of the "configurations" array use the shortcut for
autocompletion. A input box will appear that contains an entry "Neo Contract: Launch". Select it,
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

> Currently, debugging is only supported inside of the contract class. If the class makes
> calls to methods in the devpack or in other custom classes in your workspace, the debugger will
> not step into those methods.

