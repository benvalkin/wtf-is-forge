## Translations

### What are translations?
Translations are just a big config file for human readable text that gets displayed in game (with things like item names, GUIs, and chat messages), because it wouldn't be a good idea hardcode that sort of thing. Translations also allow you to add different language support so that people in other countries can read and understand it. If you've developed stuff for other games, you didn't need to read all this paragraph.

Translations for Forge mods are written in the form of a JSON dictionary that resides in a file inside `src\main\resources\assets\yourmodid\lang`.

To add English translations, make a folder in this directory called `en_us.json`. Make sure the file named correctly otherwise it will not work.

![image](https://user-images.githubusercontent.com/69928007/153729328-c76c0341-7529-468c-8d77-8b8eb864086f.png)

Here is what the file should look like. It should contain translations for all items, your mod name (`itemGroup.yourmodid`), and pretty much anything that will be displayed as text in game.

![image](https://user-images.githubusercontent.com/69928007/153729354-3d4d9e85-f90a-4809-975c-013e286811fc.png)

You might notice that for items, the translation key format is `item.modid.itemname`.
So any time you want to add a translation for a new item, you need to add a new JSON dictionary entry in the translations file of your desired language.

## Hold up, the `item.modid.itemname` format looks familiar...

If you've played modded Minecraft before, you might have seen something like this in some (not-so-polished) mods:

![image](https://user-images.githubusercontent.com/69928007/153729528-ac16b2f5-f76b-47cb-add4-c4cd3854db72.png)

This is happening because the translations file of the current language has missing keys, because the mod creator was careless and forgot to add them.
So the game tried to look for two dictionary keys called `item.agricultured.chilipepper` and `itemGroup.agricultured`, but none could be found.
Literally all you would need to do to fix this is to add these two lines to your translations file:
```json
  "itemGroup.agricultured": "Agricultured",
  "item.agricultured.chilipepper": "Chili Pepper",
```







