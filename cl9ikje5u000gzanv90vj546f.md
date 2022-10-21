# Debugging Cake on macOS with Mono

If you're not already familiar with Cake, [I wrote a post](https://redth.codes/would-you-like-some-csharp-in-your-cake/) awhile back about the deliciousness of Cake for creating your build scripts in C#.

Cake recently released `v0.22.0` which marked a huge milestone for the project. This is the first release of Cake to use Roslyn under the hood on Mono for compiling your build scripts. This is sort of bittersweet to me since helping implement mono support for Cake using `Mono.CSharp` was one of the [first contributions](https://github.com/cake-build/cake/pull/280) I made to the project, but it's definitely time to move on! It's also pretty big for windows users too since the newer version of Roslyn now used brings support for the latest and greates C# features.

Breakpoints, Locals, Variable inspection, oh my!
------------------------------------------------

Cake has been fantastic. Truly a huge time saver for me. But it's always left me wanting just a bit more. Traditionally it's been impossible to properly debug cake scripts on Mono. This means you ended up sprinkling `Information ("value={0}", value);` throughout your code when you ran into issues you were trying to solve.

Finally, we can leave this all behind us! Since Cake now uses Roslyn on Mono, we can have proper debug symbols emitted for our build scripts, and using Mono's debugger we can actually attach to the running process.

Yes, you heard it right, we now can have breakpoints, locals, variable inspection, and all the nice things in life!

The easiest way to get this up and running is to use Visual Studio Code, however you can of course attach to the process with anything that knows how to speak to the mono debugger.

Shut up and take my money!
--------------------------

Ok already, here's how to make the magic happen in VSCode:

### Prerequisites

1.  Install the [Mono Debugger VSCode extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.mono-debug).
2.  Install the [Cake VSCode extension](https://marketplace.visualstudio.com/items?itemName=cake-build.cake-vscode).
3.  Open your directory in VSCode which contains your `build.cake` script.
4.  Use the `Cake: Install a bootstrapper` command to install `build.sh` from the `View -> Command Palette` menu.
5.  Download `Cake.exe` to your `./tools/` directory - the easiest way to do this is to simply run the bootstrapper once in terminal: `sh build.sh`. (Note: this will invoke your `build.cake` script with the `Default` target).

> Currently there is a `Cake: Download Debug Dependencies` command available, but it installs `CakeCLR` which is meant to run on _dotnetcore_ not _Mono_. Hopefully future versions of the Cake extension will include a command for the Mono Debug Dependencies as well!

### Configuring launch.json

If you're not familiar with VSCode, in order to start a project, you need a `.vscode/launch.json` file configured.

The Cake VSCode extension can help us set one up, but just like before, it's meant for debugging with _dotnetcore_ and won't work for Mono.

Go ahead and create a directory `.vscode` and within it, a file `launch.json` with the contents:

    {
        "version": "0.2.0",
        "configurations": [
            {
                "name": "Cake: Debug Script (mono)",
                "type": "mono",
                "request": "launch",
                "program": "${workspaceRoot}/tools/Cake/Cake.exe",
                "args": [
                    "${workspaceRoot}/build.cake",
                    "--debug",
                    "--verbosity=diagnostic"
                ],
                "cwd": "${workspaceRoot}",
                "console": "internalConsole"
            }
        ]
    }
    

Start your debugger
-------------------

Finally, we're ready to go. Switch to the debug view (from the `View -> Debug` menu),

Open your `build.cake` file, set some breakpoints in VSCode (or you can also set them with a `#break` preprocessor directive in your build script code).

`F5` (or the `Debug -> Start Debugging` menu) and watch the magic happen!

![cake-debug](https://cdn.hashnode.com/res/hashnode/image/upload/v1666361372649/K1nWbUkQW.html)

Enjoy!