# Hook

A lightweight library for intercepting and detouring Roblox remote calls safely.

- `__namecall` (for any method called with the `:` operator, like `object:FireServer(...)`)
- `RemoteEvent.FireServer`
- `UnreliableRemoteEvent.FireServer`
- `RemoteFunction.InvokeServer`
- `Player.Kick`

The library also catches direct calls such as  
```lua
hookfunction(Instance.new('RemoteFunction').InvokeServer, ...)
```  
instead of the usual  
```lua
RemoteFunction:InvokeServer(...)
```
In other words, **any** usage of `RemoteFunction:InvokeServer` can be intercepted, even when invoked in a less typical manner.

---

## Installation

Load it in your script or exploit environment. For example:
   ```lua
   -- In your script:
   local Hook = loadstring(game:HttpGet('https://github.com/viewvector/Hook/blob/main/Hook.lua?raw=true'))()
   ```
That’s it! Once loaded, all the hooked functions are automatically replaced.

---

## How It Works

- **`MetaMethods.OriginalNameCall`**: Hooks the raw `__namecall` behind every Roblox Instance.  
- **`MetaMethods.OriginalFireServer`**: Hooks remote event calls.  
- **`MetaMethods.OriginalInvokeServer`**: Hooks remote function calls, including unusual calls such as `Instance.new('RemoteFunction').InvokeServer(remote, ...)`.  
- **`MetaMethods.OriginalUnreliableFireServer`**: Hooks “unreliable” remote events.  
- **`MetaMethods.OriginalKick`**: Hooks kick function.  

A custom function `MetaMethods.NameCall` decides how to handle each method. The variable `getgenv().namecall` is exposed so you can override how the varargs are processed before going back to the original calls.

### Getting the NameCall Method

Inside the NameCall hook, we store the current name call method in `Hook.CurrentMethod`. To retrieve it outside of the hook, you can call:

```lua
local Method = Hook:GetNameCallMethod()
```

This lets you check which method (e.g. `"FireServer"` or `"InvokeServer"`) was most recently intercepted.

---

## Usage

You can override the public `namecall` function to see or modify arguments. For instance:

```lua
-- Load the handler
local Hook = loadstring(game:HttpGet('https://github.com/viewvector/Hook/blob/main/Hook.lua?raw=true'))()

namecall = function(...)
    local method = Hook:GetNameCallMethod()
    local remote = select(1, ...)
    local args = {select(2, ...)}

    if method == "FireServer" then
        warn(("[Hook] FireServer → %s"):format(remote:GetFullName()))
        warn("[Hook] Args:", unpack(args))
    elseif method == "InvokeServer" then
        warn(("[Hook] InvokeServer → %s"):format(remote:GetFullName()))
        warn("[Hook] Args:", unpack(args))
    end

    return ...
end
```

When a script calls `SomeRemoteEvent:FireServer("Hello", 123)` or `someRemoteFunction:InvokeServer("ABC")`, you’ll see a debug print with the remote name and arguments, and the original call will still run normally.

This library also captures less common calls like:
```lua
Instance.new('RemoteFunction').InvokeServer(RemoteFunction, "xyz", 789)
```
so you can reliably intercept *any* usage pattern of `RemoteFunction`.

---

## Example

Below is a minimal usage example you might drop into your script after loading `Hook`:

```lua
-- Load the handler
local Hook = loadstring(game:HttpGet('https://github.com/viewvector/Hook/blob/main/Hook.lua?raw=true'))()

-- Example: track all 'FireServer' events
namecall = function(...)
    local method = Hook:GetNameCallMethod()
    local remote = select(1, ...)
    local args = {select(2, ...)}

    if method == "FireServer" then
        warn(("[Hook] Intercepted FireServer → %s"):format(remote and remote.Name or "Unknown"))
        warn("[Hook] Arguments:", unpack(args))
    end

    return ...
end

```

---

## Disclaimer

- Overriding NameCall can be risky if you accidentally break argument ordering. Make sure to always return the correct arguments in the correct order.
- This script is intended for *educational and debugging* purposes. Use responsibly.  
