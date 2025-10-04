# KitsuneMenu API - Hướng Dẫn Sử Dụng

### 3. Using statements

```csharp
using Menu;
using Menu.Enums;
```

---

## 🚀 Quick Start

### Khởi tạo trong plugin

```csharp
public class MyPlugin : BasePlugin
{
    private KitsuneMenu? _menuManager;

    public override void Load(bool hotReload)
    {
        // multiCast: false = chỉ 1 button tại 1 thời điểm
        _menuManager = new KitsuneMenu(this, multiCast: false);
    }
}
```

### Menu đơn giản

```csharp
[ConsoleCommand("css_simplemenu")]
public void ShowSimpleMenu(CCSPlayerController? player, CommandInfo info)
{
    if (player == null) return;

    var items = new List<MenuItem>();

    items.Add(new MenuItem(MenuItemType.Spacer));

    items.Add(new MenuItem(
        MenuItemType.Button,
        new MenuValue("Click Me "),
        [
            new MenuButtonCallback("→", "action", (ctrl, data) =>
            {
                ctrl.PrintToChat("Button clicked!");
            })
        ]
    ));

    _menuManager!.ShowScrollableMenu(
        controller: player,
        title: "Simple Menu",
        items: items,
        callback: null,
        isSubmenu: false,
        freezePlayer: false,
        visibleItems: 10
    );
}
```

---

## 📚 API Reference

### ShowScrollableMenu

Method chính để hiển thị menu.

```csharp
void ShowScrollableMenu(
    CCSPlayerController controller,      // Player nhận menu
    string title,                         // Tiêu đề menu
    List<MenuItem> items,                 // Danh sách items
    Action<MenuButtons, MenuBase, MenuItem?>? callback,  // Callback xử lý
    bool isSubmenu = false,               // Có phải submenu không
    bool freezePlayer = false,            // Đóng băng player
    int visibleItems = 5,                 // Số items hiển thị
    Dictionary<int, object>? defaultValues = null,  // Giá trị mặc định
    bool disableDeveloper = false         // Ẩn footer developer
)
```

### ShowMenu

Method cơ bản cho custom menu.

```csharp
void ShowMenu(
    CCSPlayerController controller,
    MenuBase menu,
    Action<MenuButtons, MenuBase, MenuItem?> callback,
    bool isSubmenu = false,
    bool freezePlayer = false
)
```

### Utility Methods

```csharp
// Xóa tất cả menu của player
void ClearMenus(CCSPlayerController controller)

// Đóng menu hiện tại (quay lại menu trước)
void PopMenu(CCSPlayerController controller, MenuBase? menu = null)

// Kiểm tra menu hiện tại
bool IsCurrentMenu(CCSPlayerController controller, MenuBase menu)

// Lấy stack menu
Stack<MenuBase>? GetMenus(CCSPlayerController controller)

// Set stack menu
void SetMenus(CCSPlayerController controller, Stack<MenuBase> menus)
```

---

## 🎨 MenuItem Types

### 1. Text (Chỉ hiển thị)

```csharp
items.Add(new MenuItem(
    MenuItemType.Text,
    new MenuValue("This is text")
    {
        Prefix = "<font color=\"#ff0000\">",
        Suffix = "</font>"
    }
));
```

### 2. Button (Nút bấm với callback)

```csharp
items.Add(new MenuItem(
    MenuItemType.Button,
    new MenuValue("Button Label "),
    [
        new MenuButtonCallback("Action", "data_id", (ctrl, data) =>
        {
            ctrl.PrintToChat($"Clicked: {data}");
        })
    ]
));
```

### 3. Bool (Toggle ON/OFF)

```csharp
var boolItem = new MenuItem(
    MenuItemType.Bool,
    new MenuValue("God Mode: ")
);
boolItem.Data[0] = 1; // 1 = ON, 0 = OFF
items.Add(boolItem);
```

### 4. Choice (Chọn 1 trong nhiều)

```csharp
items.Add(new MenuItem(
    MenuItemType.Choice,
    new MenuValue("Team: "),
    [
        new MenuValue("Auto"),
        new MenuValue("CT"),
        new MenuValue("T")
    ],
    pinwheel: true  // Cho phép quay vòng
));
```

### 5. ChoiceBool (Nhiều bool cùng lúc)

```csharp
var choiceItem = new MenuItem(
    MenuItemType.ChoiceBool,
    new MenuValue("Options: "),
    [
        new MenuValue("Chat"),
        new MenuValue("Sound"),
        new MenuValue("HUD")
    ]
);
choiceItem.Data[0] = 1; // Chat ON
choiceItem.Data[1] = 0; // Sound OFF
choiceItem.Data[2] = 1; // HUD ON
items.Add(choiceItem);
```

### 6. Slider (Giá trị 0-10)

```csharp
var sliderItem = new MenuItem(
    MenuItemType.Slider,
    new MenuValue("Volume: ")
);
sliderItem.Data[0] = 5; // Giá trị mặc định
items.Add(sliderItem);
```

### 7. Input (Nhập text qua chat)

```csharp
var inputItem = new MenuItem(
    MenuItemType.Input,
    new MenuValue("Enter name: ")
);
inputItem.DataString = "Default Name";
items.Add(inputItem);
```

### 8. Spacer (Khoảng trống)

```csharp
items.Add(new MenuItem(MenuItemType.Spacer));
```

---

## 🎮 Menu Callbacks

### Callback Structure

```csharp
private void OnMenuCallback(MenuButtons button, MenuBase menu, MenuItem? item)
{
    switch (button)
    {
        case MenuButtons.Select:
            // Xử lý khi bấm Select (Space)
            break;

        case MenuButtons.Up:
            // Xử lý khi bấm W
            break;

        case MenuButtons.Down:
            // Xử lý khi bấm S
            break;

        case MenuButtons.Left:
            // Xử lý khi bấm A
            break;

        case MenuButtons.Right:
            // Xử lý khi bấm D
            break;

        case MenuButtons.Back:
            // Xử lý khi bấm Shift
            break;

        case MenuButtons.Exit:
            // Xử lý khi bấm Tab
            break;

        case MenuButtons.Input:
            // Xử lý khi nhập text
            if (item?.Type == MenuItemType.Input)
            {
                string text = item.DataString;
            }
            break;
    }
}
```

### Handling Different Item Types

```csharp
private void OnMenuCallback(MenuButtons button, MenuBase menu, MenuItem? item)
{
    if (button != MenuButtons.Select || item == null) return;

    switch (item.Type)
    {
        case MenuItemType.Bool:
            bool enabled = item.Data[0] == 1;
            Server.PrintToChatAll($"Bool value: {enabled}");
            break;

        case MenuItemType.Choice:
            string selected = item.Values?[item.Option].Value ?? "";
            Server.PrintToChatAll($"Choice: {selected}");
            break;

        case MenuItemType.Slider:
            int value = item.Data[0]; // 0-10
            Server.PrintToChatAll($"Slider: {value}");
            break;

        case MenuItemType.ChoiceBool:
            bool opt1 = item.Data[0] == 1;
            bool opt2 = item.Data[1] == 1;
            bool opt3 = item.Data[2] == 1;
            break;
    }
}
```

---

## 🎨 Styling & Formatting

### Font Sizes

```csharp
// fontSize-s, fontSize-m, fontSize-l, fontSize-xl
new MenuValue("Large Text")
{
    Prefix = "<font class=\"fontSize-xl\" color=\"#00ff00\">",
    Suffix = "</font>"
}
```

---

## 💡 Examples

### Example 1: Weapon Menu

```csharp
private void ShowWeaponMenu(CCSPlayerController player)
{
    var items = new List<MenuItem>();

    string[] weapons = ["AK-47", "M4A4", "AWP", "Deagle"];
    
    foreach (var weapon in weapons)
    {
        string weaponEntity = $"weapon_{weapon.ToLower().Replace("-", "")}";
        
        items.Add(new MenuItem(
            MenuItemType.Button,
            new MenuValue($"{weapon} "),
            [
                new MenuButtonCallback("Give", weaponEntity, (ctrl, data) =>
                {
                    ctrl.GiveNamedItem(data);
                    ctrl.PrintToChat($"Given: {weapon}");
                })
            ]
        ));
    }

    _menuManager!.ShowScrollableMenu(
        player, "Weapon Menu", items, null, true, false, 8
    );
}
```

### Example 2: Settings Menu với Input

```csharp
private void ShowSettingsMenu(CCSPlayerController player)
{
    var items = new List<MenuItem>();

    // Slider
    var volumeItem = new MenuItem(MenuItemType.Slider, new MenuValue("Volume: "));
    volumeItem.Data[0] = 7;
    items.Add(volumeItem);

    // Input
    var nameItem = new MenuItem(MenuItemType.Input, new MenuValue("Name: "));
    nameItem.DataString = player.PlayerName;
    items.Add(nameItem);

    _menuManager!.ShowScrollableMenu(
        player, "Settings", items, OnSettingsCallback, true, true, 10
    );
}

private void OnSettingsCallback(MenuButtons button, MenuBase menu, MenuItem? item)
{
    if (item == null) return;

    if (button == MenuButtons.Select && item.Type == MenuItemType.Slider)
    {
        int volume = item.Data[0];
        Server.PrintToChatAll($"Volume: {volume}/10");
    }
    else if (button == MenuButtons.Input && item.Type == MenuItemType.Input)
    {
        string newName = item.DataString;
        Server.PrintToChatAll($"New name: {newName}");
    }
}
```

### Example 3: Submenu Navigation

```csharp
private void ShowMainMenu(CCSPlayerController player)
{
    var items = new List<MenuItem>();

    items.Add(new MenuItem(
        MenuItemType.Button,
        new MenuValue("Submenu 1 "),
        [
            new MenuButtonCallback("→", "sub1", (ctrl, _) => ShowSubmenu1(ctrl))
        ]
    ));

    items.Add(new MenuItem(
        MenuItemType.Button,
        new MenuValue("Submenu 2 "),
        [
            new MenuButtonCallback("→", "sub2", (ctrl, _) => ShowSubmenu2(ctrl))
        ]
    ));

    _menuManager!.ShowScrollableMenu(
        player, "Main", items, null, false, false, 10
    );
}

private void ShowSubmenu1(CCSPlayerController player)
{
    var items = new List<MenuItem>();
    // ... add items
    
    _menuManager!.ShowScrollableMenu(
        player, "Submenu 1", items, null, true, false, 10
    );
}
```

---

## 🐛 Troubleshooting

### Menu không hiển thị
- Kiểm tra player còn sống (`player.PawnIsAlive`)
- Đảm bảo có ít nhất 1 `MenuItem` trong list
- Thêm `Spacer` vào đầu/cuối menu

### Không scroll được
- Kiểm tra `menu_config.jsonc` có đúng mapping W/S
- Restart server sau khi đổi config
- Đảm bảo `visibleItems` < tổng số items

### Input không hoạt động
- Callback phải xử lý `MenuButtons.Input`
- Item type phải là `MenuItemType.Input`
- Player phải gõ vào chat (say/say_team)

### Button callback không chạy
- Khi dùng `MenuButtonCallback`, set `callback: null`
- Kiểm tra lambda function có đúng signature
- Debug bằng `Console.WriteLine` trong callback