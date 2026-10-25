# Obenseuer-BankInventory

A BepInEx 5 mod for [Obenseuer](https://store.steampowered.com/app/1554640/Obenseuer/) that adds a **bank inventory**: press a key to open a storage UI where you can deposit and take items, with **infinite storage** (base capacity, auto-expanding, multi-page) and **full save support**.

## Features

- Open/close the bank with a hotkey (default `B`).
- Deposit items from your inventory by dragging them into the bank (or use the game's "take all" button to withdraw everything).
- **Infinite storage**: starts with 30 slots; whenever the storage becomes full it automatically grows by one page. You can never lose items to the floor because there is always at least one free slot.
- **Multiple pages**: the UI shows 88 slots per page; navigate with `[` / `]` (previous / next page).
- **Save support**: the bank is a global savable object (`Storage.global = true`), serialized into `Globals.tnmt` by the game's save system. It survives scene changes and save/load cycles.
- Everything is configurable in the BepInEx config file.

## Requirements

- [BepInEx 5](https://github.com/BepInEx/BepInEx) (x64) installed in the game folder
- Obenseuer (Windows, x64, .NET Framework 4.7.2 / net472 runtime)

## Installation

1. Build the mod (see below) or grab the release DLL.
2. Copy `Obenseuer-BankInventory.dll` into `BepInEx/plugins/` inside your Obenseuer folder.
3. Launch the game. Press `B` to open/close the bank.

## Configuration

The config file `BepInEx/config/com.juanberta.obenseuer.bankinventory.cfg` is generated on first run:

| Section | Key | Default | Description |
| --- | --- | --- | --- |
| General | `OpenBankKey` | `B` | Key that opens/closes the bank inventory |
| General | `NextPageKey` | `]` (RightBracket) | Key to go to the next bank page |
| General | `PreviousPageKey` | `[` (LeftBracket) | Key to go to the previous bank page |
| Storage | `BaseCapacity` | `30` | Initial number of slots. The bank expands automatically when it becomes full |
| Storage | `PageSize` | `88` | Slots shown per page (the game UI grid supports 88 slots) |

## Building

```powershell
& "C:\Program Files\Microsoft Visual Studio\18\Community\MSBuild\Current\Bin\MSBuild.exe" Obenseuer-BankInventory.csproj /p:Configuration=Release /p:Platform=x64
```

Output: `bin\Release\Obenseuer-BankInventory.dll`

The project targets **net472 / x64 / LangVersion 7.3** (BepInEx 5 style) and references the game assemblies from `C:\Juegos\Obenseuer\Obenseuer_Data\Managed\` and BepInEx core from `C:\Juegos\Obenseuer\BepInEx\core\`. Adjust the HintPaths if your game is installed elsewhere.

## How it works (design notes)

- `BankStorage` extends the game's `Storage` component. It reuses the game's own slot/storage machinery (`StorageSlot`, `SlotController`, stacking, dragging), so all vanilla behaviors (stack merging, item meta, perishables...) work out of the box.
- **Infinite growth**: `Update()` checks `Storage.IsStorageFull()` while the bank is open. When the storage is full, `ExpandByPage()` appends one page of fresh slots and refreshes every controller's `Siblings` array so new stacks can spread into the new slots. Because the storage is never actually full during play, the game's "drop to floor" overflow path is never hit when dragging items in.
- **Paging**: all slots get a slot UI created once when the bank opens; the current page's slots are `SetActive(true)` and the others are hidden. Hidden slots still keep their UI objects, which keeps vanilla code that iterates all slots (e.g. the "Take All" button) null-safe. Pages are switched with `[` / `]`, and the panel title shows `Page X/Y`.
- **Saving**: the bank GameObject is `DontDestroyOnLoad` and registered into `SaveController.DontDestroyOnLoadObjects`. It overrides `OnSavingGameSpecial()` / `OnLoadingGameSpecial()` to serialize via `SaveController.SerializeData/DeSerializeData(this, global=true)`, so its contents live in the global save (`Globals.tnmt`) and survive scene changes and save/load cycles.

## License

MIT
