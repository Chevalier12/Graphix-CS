# Graphix-CS

Graphix-CS is a packaging fork of [SDL3-CS](https://github.com/edwardgushchin/SDL3-CS), based on upstream `v3.4.16.0` (`c1d1cb0da632cb51799da6989f6e48c52f2a539e`). The binding and generator source, namespace `SDL3`, class `SDL`, and assembly `SDL3-CS.dll` are unchanged. Original authorship and the zlib license are preserved.

This fork distributes `Graphix-CS.3.4.16.nupkg` through verified CI artifacts, not NuGet.org. See the [fork README](https://github.com/Chevalier12/Graphix-CS#fork-distribution) for local-feed installation and provenance pinning. Do not reference both `Graphix-CS` and `SDL3-CS` in one application.

The package contains no native runtime. Cerneala supplies `Graphix.Native 3.4.16-graphix.2` separately. The native package families and documentation below refer to the upstream SDL3-CS project; they are not renamed by this fork.

## Package Versions

This package uses managed version `Graphix-CS 3.4.16.0` (NuGet-normalized version `3.4.16`).

| Package family | Version |
|----------------|---------|
| `Graphix-CS` | `3.4.16` |
| `SDL3-CS.<Platform>` | `3.4.16.0` |
| `SDL3-CS.<Platform>.Image` | `3.4.6.9` |
| `SDL3-CS.<Platform>.Mixer` | `3.2.4.11` |
| `SDL3-CS.<Platform>.TTF` | `3.2.2.11` |
| `SDL3-CS.<Platform>.Shadercross` | `3.0.0.11` |

## Documentation

- [SDL3-CS Wiki](https://github.com/edwardgushchin/SDL3-CS/wiki)
- [SDL3-CS API Reference](https://github.com/edwardgushchin/SDL3-CS/wiki/API-Reference)
- [SDL3 upstream wiki](https://wiki.libsdl.org/SDL3/FrontPage)
- [GitHub Releases](https://github.com/edwardgushchin/SDL3-CS/releases)

## Supported Platforms

The managed wrapper targets .NET 7, .NET 8, .NET 9, and .NET 10.

| Platform family | Native package suffix | Supported RIDs / ABIs | Notes |
|-----------------|-----------------------|------------------------|-------|
| Windows | `Windows` | `win-x86`, `win-x64`, `win-arm64` | Dynamic SDL libraries for desktop Windows apps. |
| Linux | `Linux` | `linux-x64`, `linux-arm64` | Built against glibc 2.28 or newer. |
| macOS | `MacOS` | `osx-x64`, `osx-arm64` | Dynamic SDL libraries for Intel and Apple Silicon macOS apps. |
| Android | `Android` | `android-arm`, `android-arm64`, `android-x86`, `android-x64` | Includes the SDL Android bridge in `SDL3-CS.Android`. |
| iOS | `iOS` | `ios-arm64`, `iossimulator-arm64`, `iossimulator-x64` | Static native assets are linked through package `buildTransitive` targets. |
| tvOS | `tvOS` | `tvos-arm64`, `tvossimulator-arm64`, `tvossimulator-x64` | Static native assets are linked through package `buildTransitive` targets. |

## Installation

Install the managed wrapper after configuring the extracted artifact directory as a local NuGet source:

```bash
dotnet add package Graphix-CS --version 3.4.16
```

Add the native package family that matches your target platform. For a Windows desktop app:

```bash
dotnet add package SDL3-CS.Windows
dotnet add package SDL3-CS.Windows.Image
dotnet add package SDL3-CS.Windows.TTF
dotnet add package SDL3-CS.Windows.Mixer
dotnet add package SDL3-CS.Windows.Shadercross
```

Replace `Windows` with `Linux`, `MacOS`, `Android`, `iOS`, or `tvOS`.

| Platform | SDL | SDL_image | SDL_ttf | SDL_mixer | SDL_shadercross |
|----------|-----|-----------|---------|-----------|-----------------|
| Windows | `SDL3-CS.Windows` | `SDL3-CS.Windows.Image` | `SDL3-CS.Windows.TTF` | `SDL3-CS.Windows.Mixer` | `SDL3-CS.Windows.Shadercross` |
| Linux | `SDL3-CS.Linux` | `SDL3-CS.Linux.Image` | `SDL3-CS.Linux.TTF` | `SDL3-CS.Linux.Mixer` | `SDL3-CS.Linux.Shadercross` |
| macOS | `SDL3-CS.MacOS` | `SDL3-CS.MacOS.Image` | `SDL3-CS.MacOS.TTF` | `SDL3-CS.MacOS.Mixer` | `SDL3-CS.MacOS.Shadercross` |
| Android | `SDL3-CS.Android` | `SDL3-CS.Android.Image` | `SDL3-CS.Android.TTF` | `SDL3-CS.Android.Mixer` | `SDL3-CS.Android.Shadercross` |
| iOS | `SDL3-CS.iOS` | `SDL3-CS.iOS.Image` | `SDL3-CS.iOS.TTF` | `SDL3-CS.iOS.Mixer` | `SDL3-CS.iOS.Shadercross` |
| tvOS | `SDL3-CS.tvOS` | `SDL3-CS.tvOS.Image` | `SDL3-CS.tvOS.TTF` | `SDL3-CS.tvOS.Mixer` | `SDL3-CS.tvOS.Shadercross` |

Android applications use `MainActivity : Org.Libsdl.App.SDLActivity`, override `GetLibraries()`, and run SDL from the managed `Main()` override.

## Managed Main Callbacks

For SDL's callback-based application lifecycle, implement `SDL.IMainCallbacks<TSelf>` and apply `[SDL.GenerateMain]` to the partial application class:

```csharp
using SDL3;

[SDL.GenerateMain]
internal sealed partial class Game : SDL.IMainCallbacks<Game>
{
    public static SDL.AppResult AppInit(out Game? appState, string[] args)
    {
        appState = new Game();
        return SDL.AppResult.Continue;
    }

    public SDL.AppResult AppIterate() => SDL.AppResult.Continue;

    public SDL.AppResult AppEvent(ref SDL.Event @event) =>
        (SDL.EventType)@event.Type == SDL.EventType.Quit
            ? SDL.AppResult.Success
            : SDL.AppResult.Continue;

    public void AppQuit(SDL.AppResult result)
    {
    }
}
```

`Graphix-CS` supplies the unchanged SDL3-CS source generator through the package's analyzer assets. It creates the entry point and delegates to `SDL.RunMainCallbacks<TApp>`, which owns the managed/native state lifetime and contains managed exceptions until SDL returns control to the caller.

## Example

```csharp
using SDL3;

namespace CreateWindow;

internal static class Program
{
    [STAThread]
    private static void Main()
    {
        if (!SDL.Init(SDL.InitFlags.Video))
        {
            SDL.LogError(SDL.LogCategory.System, $"SDL could not initialize: {SDL.GetError()}");
            return;
        }

        if (!SDL.CreateWindowAndRenderer("SDL3 Create Window", 800, 600, 0, out var window, out var renderer))
        {
            SDL.LogError(SDL.LogCategory.Application, $"Error creating window and renderer: {SDL.GetError()}");
            return;
        }

        SDL.SetRenderDrawColor(renderer, 100, 149, 237, 255);
        SDL.RenderClear(renderer);
        SDL.RenderPresent(renderer);

        SDL.DestroyRenderer(renderer);
        SDL.DestroyWindow(window);
        SDL.Quit();
    }
}
```

## Feedback and Contributions

Open an [issue](https://github.com/edwardgushchin/SDL3-CS/issues) or start a [discussion](https://github.com/edwardgushchin/SDL3-CS/discussions) for bugs, ideas, and questions.

See the [repository README](https://github.com/edwardgushchin/SDL3-CS#readme) for the full project overview.
