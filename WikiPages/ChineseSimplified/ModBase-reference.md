## The ModBase class

HugsLib提供的最简单的存取“工具”和“事件”的方法，是创建一个继承自`HugsLib.ModBase`的类。
如果想要在Visual Studio使用HugsLib中的类型，你必须给你的项目添加一个引用。也推荐你添加[checker assembly]程序集(https://github.com/UnlimitedHugs/RimworldHugsLibChecker)，来确保Mod加载时，该类库已经安装。
>The easiest way to access the facilities and events HugsLib provides is to create a class that extends `HugsLib.ModBase`.
To access the type in Visual Studio the HugsLib dll must be added as a reference to your project. It's also recommended to add add the [checker assembly](https://github.com/UnlimitedHugs/RimworldHugsLibChecker) to your mod to make sure the library is installed when the mod is loaded.

如果一个类型继承了`ModBase`，那么就会被HugsLib在游戏开始自动实例化。确保你写的类有一个无参数的构造函数，你也可以把它标为`private`。
>When a type extends ModBase, it will be automatically instantiated by HugsLib on game start. Make sure that your class has a parameterless constructor, which can be private if you prefer.

***

### Properties

### ModIdentifier
每个ModBase的继承类都需要一个独一无二的标识符。可以通过重写`ModIdentifier`属性来为你的Mod提供它。该标识符将在“设置”的XML文件中用来储存你的“设置”，所以要避免使用空格和其他特殊字符。如果你提供了一个不正确的标识符就会收到一个异常提示。
>Each ModBase class needs to have a unique identifier. Provide yours by overriding the `ModIdentifier` property. The identifier will be used in the settings XML to store your settings, so avoid spaces and special characters. You will get an exception if you provide an improper identifier.

### Logger
Logger属性允许Mod在控制台输出可识别的消息。`Error`和`Warning`方法都是可用的。调用方法：
`Logger.Message("test");`
会使控制台输出下面的字符：
`[ModIdentifier] test` 
此外，Logger属性的`Trace`方法只会当处于开发者模式时在控制台显示信息。
>The Logger property allows a mod to write identifiable messages to the console. Error and Warning methods are also available. Calling:  
`Logger.Message("test");`  
will result in the following console output:  
`[ModIdentifier] test`  
Additionally, the Trace method of the logger will write a console message only if Rimworld is in Dev mode.

### ModIsActive
如果Mod可以在“Mod配置”对话框中找到，就会返回`true`。禁用的Mod不会被加载或实例化，如果一个Mod是可用的，却在“Mod配置”被禁用，该属性就会返回`false`。
>Returns true if the mod is enabled in the Mods dialog. Disabled mods would not be loaded or instantiated, but if a mod was enabled, and then disabled in the Mods dialog this property will return false.

### Settings
为你的Mod返回`ModSettingsPack`类型的值，在该类型中你可以获得`List<SettingHandle>`类型的“handles”。更多信息请查看“Adding-mod-settings”页面。
>Returns the `ModSettingsPack` for your mod, from where you can get your `SettingsHandle`s. See the wiki page of creating configurable settings for more information.

***

## Events
从HugsLib控制器接收事件，继承类必须重写`ModBase`提供的方法。
下面是可以被接收的事件：
>To receive events from the HugsLib controller the extending class must override the methods provided by `ModBase`. The following are the events that can be received:

### Initialize
在Mod初始化时被通知一次，紧接实例化之后。
如果Mod的配置改变，或者Def被重新加载，该方法不会再被调用。
>Called once when the mod is first initialized, closely after it is instantiated.  
If the mods configuration changes, or Defs are reloaded, this method is **not** called again.

### DefsLoaded
在Def加载后被通知。
这发生在游戏开始时，也发生在Def重新载入和Mod配置改变的时候。这时候适合注入程序性的Def。确保你调用了`HugsLib.InjectedDefHasher.GiveShortHasToDef`，这可以在你手动实例化时避免Def冲突（这是很常见的事）。
DefsLoaded也推荐用来放置你的“settings handles”。理由是，当玩家改变了游戏的语言，`DefsLoaded`就会被调用，“settings handles”也因此会载入正确的语言文本。
牢记，当你的Mod在“Mod配置”对话框中被禁用，`DefsLoaded`仍然会被调用。你可以使用`ModIsActive`属性，如果你想解决这个问题。
>Called after all Defs are loaded.  
This happens when the game is started, as well as when Defs are reloaded and the mod configuration changes. This is a good time to inject any procedural defs. Make sure you call `HugsLib.InjectedDefHasher.GiveShortHasToDef` on any defs you manually instantiate to avoid def collisions (it's a vanilla thing).  
DefsLoaded is also the recommended place to get your settings handles. The reason being, that when the player changes the game language, `DefsLoaded` will be called, and the settings handles will be updated with strings in the proper language.  
Keep in mind, that `DefsLoaded` will be called even after your mod is disabled in the Mods dialog. Use the `ModIsActive` property if you need to account for that.

### Update
每一帧都会被通知。
牢记游戏帧数的速率显然是不同的，所以这个回调函数推荐只在绘制自定义控件时使用。
>Called on each frame.  
Keep in mind that frame rate varies significantly, so this callback is recommended only to do any custom drawing.

### FixedUpdate
每次Unity更新物理数据时就会被通知。
这和`Update`有点类似，但和帧数是无关的。
>Called on each physics update by Unity.  
This is like `Update`, but independent of frame rate.

### OnGUI
当Unity GUI系统被刷新或接受一个出入事件时就会被通知。
这时候适合绘制自定义的GUI外观和控件。
同样对于监听输入事件是有用的，例如按下某些按键。下面是一个键盘监听器的例子。
>Called when the Unity GUI system is redrawn or receives an input event.  
This is a good time to draw custom GUI overlays and controls.  
Also useful for listening for input events, such as key strokes. Here's an example of a key binding listener:

```cs
if (Event.current.type == EventType.KeyDown) {
	if (KeyBindingDefOf.Misc1.JustPressed) {
		// do things
	}
}
```

### SceneLoaded
当Unity场景改变后就会被通知。接受一个`UnityEngine.SceneManagement.Scene`类型的参数。
RimWorld中有2个场景——`Entry`和`Play`，一个代表菜单，一个代表游戏本身。使用`Verse.GenScene`来检查哪个场景已经被加载。
注意，不是所有的东西都会在场景改编后初始化，而且游戏可能在地图加载或创建的中间。
>Called after a Unity scene change. Receives a `UnityEngine.SceneManagement.Scene` type argument.  
There are two scenes in Rimworld- `Entry` and `Play`, which stand for the menu, and the game itself. Use `Verse.GenScene` to check which scene has been loaded.  
Note, that not everything may be initialized after the scene change, and the game may be in the middle of map loading or generation.

### WorldLoaded
在游戏开始并且世界已经初始化后被通知。
此时，可能没有地图被初始化。
这是一个极佳的位置，用来取得你储存在文件中的`UtilityWorldObject`类型数据。在对应的Wiki页面中查看它的使用方法。
这只在游戏开始时通知，而不是在“选择着陆点”的时候。
>Called after the game has started and the world has been initialized.  
Any maps may not have been initialized at this point.  
This is a good place to get your `UtilityWorldObject`s with the data you store in the save file. See the appropriate wiki page on how to use those.  
This is only called after the game has started, not on the "select landing spot" world map.

### MapComponentsInitializing
在初始化地图时被通知，更确切的说法是在`Verse.Map.ConstructComponents()`方法调用时。接受一个`Verse.Map`类型的参数。
这是一个好位置来做一些偷偷摸摸的事情，或者操作一些地图加载完就无法获得的数据。
在地图被文件中的数据充填之前，这些做法是合适的。
>Called during the initialization of a map, more exactly during `Verse.Map.ConstructComponents()`. Receives a `Verse.Map` type argument.  
This is a good place for sneaky business and getting access to data that is unavailable after map loading has completed.  
This is right before the map is populated with data from a save file. 

### MapLoaded
在地图加载生成并且`Verse.MapDrawer.RegenerateEverythingNow`执行完成之后通知。接受一个`Verse.Map`类型的参数。
这是个好的位置来运行针对游戏地图的初始化代码。
注意，这个方法可能在加载存档后调用0次或多次，这取决于玩家在这时激活了多少个地图。
>Called after map loading and generation is complete and after `Verse.MapDrawer.RegenerateEverythingNow` was executed. Receives a `Verse.Map` type argument.  
This is a good place to run initialization code specific to a game map.  
Note, that this method may be called zero or multiple times after loading a save, depending on how many maps the player has active at the moment.

### Tick
当游戏已经加载，每次计时都会被通知。接受一个`int`类型的参数，该参数是当前的计时数。
甚至当玩家已经处于世界地图界面，而且没有地图被加载时也会通知调用。
但是不会在“选择着陆点”页面通知调用。
>Called during each tick, when a game is loaded. Receives an `int` argument, which is the number of the current tick.  
Will be called even if the player is on the world map and no map is currently loaded.  
Will not be called on the "select landing spot" world map.

### SettingsChanged
在玩家改变了某些设置并且关闭“Mod设置”对话框时通知调用。
注意，这也会因为其他Mod的设置更改而触发。
>Called after the player closes the Mod Settings dialog after changing any setting.  
Note, that the setting changed may belong to another mod.