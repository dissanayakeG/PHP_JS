# Linux Mint Customization Guide (Cinnamon Edition)

Customizing Linux Mint allows you to transform the desktop environment to match your workflow and aesthetic preferences. Follow these steps to set up your system.

## 1. Desktop & Panel Adjustments

 Desktop Backgrounds & Desklets: Right-click anywhere on the desktop to change the wallpaper or add Desklets (widgets like clocks, system monitors, or weather).
 Panel Configuration:  Right-click the panel and select Panel settings to change its height or auto-hide behavior.
 To move the panel (e.g., to the top), right-click the panel -> Modify panel -> Move panel.
 Applets: Right-click the panel -> Applets to add functional icons like CPU monitors, "Cava" visualizers, or weather.

## 2. Installing Themes and Icons

Themes and icons can be installed for a single user or globally for all users.

### Directory Locations

| Scope | Icons Directory | Themes Directory |
| --- | --- | --- |
| Local (Current User) | `~/.icons` | `~/.themes` |
| Global (All Users) | `/usr/share/icons` | `/usr/share/themes` |

> Note: If the `~/.icons` or `~/.themes` folders do not exist in your Home directory, you can simply create them.

### Applying the Theme

1. Go to Menu -> System Settings -> Themes.
2. Mix and match your choices for Mouse Pointer, Applications (GTK), Icons, and Desktop (Shell).

## 3. Step-by-Step Example: Manual Installation

### Installing the "Jasper" GTK Theme

1. Visit [Cinnamon-look.org](https://www.cinnamon-look.org).
2. Search for Jasper-gtk-theme, go to the Files tab, and download the `.tar.xz` file.
3. Extract the file. You will see several folders (e.g., Jasper-Light, Jasper-Dark).
4. Move these sub-folders into `~/.themes`.
5. Open Themes settings and select "Jasper" under the Applications category.

### Installing "Reversal" Icons

1. Search for Reversal icon theme on Cinnamon-look.
2. Download and extract the archive.
3. Move the resulting folders into `~/.icons`.
4. Open Themes settings and select "Reversal" under the Icons category.

## 4. Pro-Tips for a Better Look

 Transparency: To get a transparent panel, go to Themes -> Add/Remove and download the "Transparent Panel" extension.
 The "Spices" Store: Instead of manual downloading, you can use the Add/Remove tab inside the Themes, Applets, or Desklets settings to browse the official Linux Mint "Spices" repository.
 Terminal Transparency: Open the Terminal, go to Edit -> Preferences -> Profiles -> Colors and check "Use transparent background."
 System Fonts: Go to System Settings -> Font Selection to change the system-wide typography.

## 5. Essential Resources

 [Cinnamon Spices](https://cinnamon-spices.linuxmint.com/): Official themes, applets, and desklets.
 [Gnome-look.org](https://www.gnome-look.org): The largest repository for GTK themes and icons.

### Linux Mint (Cinnamon Edition) does not use GNOME Shell Extensions
It is important to clarify a key technical point first: Linux Mint (Cinnamon Edition) does not use GNOME Shell Extensions.
Because Cinnamon was forked from GNOME years ago, it now has its own system. Using "GNOME Tweaks" or
"GNOME Shell Extensions" on a standard Linux Mint Cinnamon desktop won't work and can sometimes cause software
conflicts.

## 5. Adding "Spices" (Cinnamon's Version of Extensions)

In Linux Mint, what GNOME calls "Extensions," Cinnamon calls Spices. You don't need a browser extension for this,
it is built directly into your settings.

### How to add/remove "Simple Net Speed":

1. Go to Menu -> System Settings.
2. Under the "Preferences" section, click on Applets.
3. Click the Download tab and click "Yes" to update the cache.
4. Search for "Simple net speed".
5. Click the Download arrow next to it.
6. Switch back to the Manage tab, select it, and click the (+) Plus icon to add it to your panel.
> To remove: Simply select it in the "Manage" tab and click the (-) Minus icon.

## 6. GNOME Tweaks vs. Cinnamon Settings

Many online guides tell you to install `gnome-tweaks`, but here is why you usually don't need it on Mint:
| Feature | GNOME Tweaks (For GNOME/Ubuntu) | Cinnamon Settings (For Mint) |
| --- | --- | --- |
| Purpose | Unlocks hidden settings in GNOME. | Built-in, central hub for all settings. |
| Themes | Required to change Shell themes. | Built-in under "Themes." |
| Extensions | Managed via a separate browser tool. | Managed via "Applets/Extensions" in Settings. |
| Window Buttons | Used to move buttons to the left. | Built-in under "Windows" -> "Button layout." |

> Recommendation: Avoid using GNOME Tweaks on Linux Mint Cinnamon. It is designed for a different desktop environment. Instead, use the native System Settings app, which is much more powerful out of the box.

## Summary of Terms (The Difference)

 Applets: Small tools that live on your panel (Clock, Net Speed, Weather).
 Desklets: Small tools that live on your desktop wallpaper (Sticky notes, Photo frames).
 Extensions (Cinnamon): These change how the system behaves (e.g., adding transparency effects or changing window tiling).
 GNOME Shell Extensions: Do not use these. They are for the GNOME desktop only and are incompatible with Cinnamon.

 ## 7. Installing custom fonts
- Go to Google Fonts.
- Search for "Inter"
- Download it and extract
- Place extracted font files in the ~/.local/share/fonts (create if the folder doesn't exist).
- Search for "Font selection" in the menu.
- Select the Inter

>Update the font cache so apps recognize them -> fc-cache -f -v