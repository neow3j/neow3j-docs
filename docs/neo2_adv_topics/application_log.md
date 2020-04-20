# Reading Application Logs

After the invocation of a smart contract, one can retrieve the results of that invocation with the `getapplicationlog`
RPC method. It contains the result stack from the VM and any notifications that the contract has uttered while
executing. In neow3j both types of information are available as `StackItem`s.

There are multiple subclasses of `StackItem`, each related to one of the possible stack item types. These are byte
array, integer, boolean, array, map, and struct. Neow3j deserializes the JSON result coming from the `getapplicationlog`
call to the correct type. But the different types can't all be handled under the same interface, and therefore you will
find that in some cases you still need to check for the type of the `StackItem` to know how to read it correctly. We'll
explain this with the example below.

With the ID of an invocation transaction, you can get its application log as follows.

```java
Neow3j neow3j = Neow3j.build(new HttpService("https://node2.neocompiler.io"));

NeoApplicationLog applicationLog = neow3j
        .getApplicationLog("c141087aee1ef13ba5c4a953e57d544e20a0ea3b4c175a4a0ef408a744e7ef67")
        .send()
        .getApplicationLog();
```

In this example the contract invocation caused one execution of the contract, so we get the first execution object and
inspect the result stack.

```java
Execution exec =  applicationLog.getExecutions().get(0);
StackItem result = exec.getStack().get(0);
if (result instanceof ByteArrayStackItem) {
    System.out.println("Execution result: " + result.asByteArray().getAsString());
} else if (result instanceof IntegerStackItem || result instanceof BooleanStackItem) {
    System.out.println("Execution result: " + result.getValue());
}
```

The example shows that the result needs to be handled specifically if it is of the type byte array. The problem with
byte array values is, that they can be interpreted in different ways. In the example, the bytes are read as a string
(`ByteArrayStackItem.getAsString()`). Neow3j cannot predict how the bytes are encoded, and so the developer needs to
know what type the smart contract will return. If the return type is an integer or boolean, the same method
(`StackItem.getValue()`) can be used to retrieve the value. The `result` object is already of the correct subtype and
the method will, therefore, return a `Boolean` or a `BigInteger`. What is not covered in the example are the remaining
return types array, map, and struct. These would also need special handling.

Next to the return type, we can also inspect the notifications in just the same way.

```java
System.out.println("Notifications:");
for (Notification notification : exec.getNotifications()) {
    StackItem item = notification.getState();
    if (item instanceof ByteArrayStackItem) {
        System.out.println("\t" + item.asByteArray().getAsAddress());
    } else if (item instanceof IntegerStackItem || item instanceof BooleanStackItem) {
        System.out.println("\t" + item.getValue());
    }
}
```

Again, to make sense of the logs, the developer needs to have some information about the types of notification. 

If you need the `StackItem` in its subclass you can use one of the type-casting methods. For example
`StackItem.asByteArray()` as shown in the example.
