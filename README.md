# Un-debugged Firefox

Obfuscated JavaScript files often use a technique called debugger traps to prevent you from reverse engineering the behavior of a script by calling

```js
while (true) {
  debugger;
}
```

when they detect devtools opening. This is trivial to get around by simply disabling breakpoints in your code, but with that approach you don't get to use actual breakpoints which makes life difficult.

---

This patch makes `debugger;` behave like `break;` so infinite loops are instantly broken out of.

In order to use the actual `debugger;` statement, you can either use browser breakpoints, or the keyword `xdebugger;`

## Usage

This patch requires you to build firefox from scratch which is going to be pretty slow.

Follow through with the build tutorial here https://firefox-source-docs.mozilla.org/setup/linux_build.html

Once you've gone through the steps required to cloned the repo from mercurial, copy the patch file to the repository root and run

`hg import debugger.patch --no-commit`

and compile firefox using `./mach compile` at the repository root.

## Limitations

This can cause problems when debugger is called with `eval("debugger")` since eval statements are evaluated in a separate context outside of loops. For some reason this doesn't break anything for me when testing with code generated through obfuscate.io, so I'm not sure how much of a limitation this actually is.

If `break` is causing issues for your use case, you can try mapping `debugger;` to `null;` instead by replacing

```cpp
return breakStatement(yieldHandling);
```

with

```cpp
return handler_.newNullLiteral(pos());
```
