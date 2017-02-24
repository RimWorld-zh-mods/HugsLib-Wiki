HugsLib的主要特点之一是可以为Mod增加一些持续性的并且可以被玩家修改的“设置”。这些“设置”的对话框可以在游戏内的“选项”->“Mod设置”找到。
>One of the main features of HugsLib is the ability for mods to add persistent settings that can be changed by the player. The settings dialog is found in Options > Mod Settings.

“设置”可以传入多种类型参数——本类库已经内置了`bool`，`int`，`string`和`Enum`类型的控件。其他的类型将被当作`string`处理，除非`CustomDrawer`被使用（详情见下）；
添加“设置”的最简单途径是定义一个继承自`HugsLib.ModBase`的类。所有的继承类都会有`Settings`属性，该属性包括了为Mod而写的`SettingsHandle`。
注意，“设置”的值通过`Name`标识符，以字符串的形式储存。在Mod定义这些“设置”之前，它们的类型是未知的。
“设置”的XML文件储存在用户的配置文件夹中。Windows系统，A16的默认路径是“`%USERPROFILE%/AppData/LocalLow/Ludeon Studios/RimWorld by Ludeon Studios/HugsLib/ModSettings.xml`”。注意，“设置”的默认值不会储存在该文件中。
>Settings can come in a variety of types- the library has built-in controls for `bool`, `int`, `string` and `Enum`. Every other type will be treated as `string`, unless `CustomDrawer` is used (see below);  
The easiest approach to adding settings is defining a class that extends `HugsLib.ModBase`. All extending classes have the property `Settings` that contains the `SettingsHandle`s for that mod.  
Note, that setting values are stored as strings by their `Name` identifier, and have no known type until they are claimed by the mod that defined them.  
The settings XML file is stored in the user profile folder. The default location on a Windows system on A16 is `%USERPROFILE%/AppData/LocalLow/Ludeon Studios/RimWorld by Ludeon Studios/HugsLib/ModSettings.xml`. Note, that settings matching their default value are not saved to the file.

***

## Basic usage
检索和创建“设置”的值，是使用同一方法完成的。使用该方法时必须为“Mod设置”对话框提供一个标题和描述（可选）。这最好在`ModBase.DefsLoaded`回调期间完成，因为这会在玩家改变游戏语言后，给“设置操作”提供标题和说明的正确翻译文本。
每个“设置”只需要被请求一次，因为当设置改变，`SettingHandle`会自动更新新的值并返回。
本示例创建了一个`bool`类型的**设置开关**：
>Retrieving and creating a setting value is done using the same method. In doing so a title and optional description must be provided for the Mod Settings dialog. This is best done during the `ModBase.DefsLoaded` callback, because this will give the setting handle the properly translated title and description after the player changes the game language.  
Each setting only needs to be requested once, because the `SettingHandle` returned will be automatically updated with new values as the setting changes.  
This example creates a boolean toggle setting:

```cs
public class SettingTest : ModBase {
	public override string ModIdentifier {
		get { return "SettingTest"; }
	}
	private SettingHandle<bool> toggle;
	public override void DefsLoaded() {
		toggle = Settings.GetHandle<bool>("myToggle", "toggleSetting_title".Translate(), "toggleSetting_desc".Translate(), false);
	}
}
```

1. `Settings.GetHandle<bool>`方法中，第一个参数是该“设置”的一个独有名字，同时是一个储存在设置XML文件里的标识符。
2. 第二个参数是“设置”的标题，它会显示在“设置对话框”的控件旁边。标题最好简洁明了，这样玩家就可以快速浏览整个菜单。“设置”具体的作用应该在说明中解释。
3. 第三个参数是“设置”的说明，当你的鼠标悬停在“设置对话框”的标题或控件上，它会弹出一个提示文本框来显示这些信息。说明可以是空的，这样就不会弹出提示文本框了。
4. 第四个参数是“设置”的默认值，即“设置”的初始值，“设置”中可以重新设置的值，也是在“设置”因为某些原因无法加载时的一条退路。

>1. The first parameter is a unique name for that setting, and is the identifier it is stored under in the settings XML file.  
>2. The second parameter is the title that will display next to the control in the settings dialog. It is best to keep the title short so that the player has an easier time skimming through the menu. What the setting does should be explained in the description.  
>3. The third parameter is the setting description, displayed as a tooltip when hovering over the setting title or control in the settings dialog. The description can be `null`, in which case the tooltip will not be displayed.  
>4. The fourth parameter is the default value, which is the starting value of the setting, the value the setting will be reset to, and the fallback value if the setting fails to load for any reason.  

使用“`string.Translate`”来注入标题和说明不是强制的，但这是一种很好的做法。PS：如果不加，翻译组就没法讲翻译好的文本替换注入。
>Using `string.Translate` to populate the title and description is optional, but a good practice.

`GetHandle`方法返回一个`SettingHandle`对象，该对象可用来存取“设置”的值并且应用其他的更改。
注意，`SettingHandle`可以隐式转换成其中储存值的类型，所以下面的两种方式读取“设置”的值是等价的：
>The `GetHandle` method returns a `SettingHandle` object that can be used to access the value of the setting and apply additional modifiers to it.  
Note, that `SettingHandle`s can be implicitly converted to the type of value they store, so the following ways of reading a setting value are equivalent:

```cs
if(toggle.Value) // do stuff

if(toggle) // do stuff
```

`int`和`string`类型的“设置”，它们的创建和存取方式是相同的，并且有各自的输入控件。`float`类型的“设置”会使用默认的文本框控件。
>Settings of type `int` and `string` are created and accessed in the same manner, and have their own input controls.  `float` settings will use the default text field control.

***

## Enum settings
`Enum`类型的“设置”是很有用的，它可以创建一种在多种状态下切换的“设置”。这些设置需要一个字符串类型的前缀标识符，用来以易读的格式显示设置选项。
下面的例子创建了一个有着3个选项的枚举类型设置：
>`Enum` settings can be quite useful to create settings that can be switched between multiple states. These settings require a string identifier prefix that will be used to display the setting options in a readable format.  
The following example creates an enum setting with 3 options:

```cs
private enum HandleEnum { DefaultValue, ValueOne, ValueTwo }
public override void DefsLoaded() {
	var enumHandle = Settings.GetHandle("enumThing", "enumSetting_title".Translate(), "enumSetting_desc".Translate(), HandleEnum.DefaultValue, null, "enumSetting_");
}
```

上面这段代码创建了一个“设置”，它需要在语言XML文件中提供下列字符串：“`enumSetting_title`，`enumSetting_desc`，`enumSetting_DefaultValue`，`enumSetting_ValueOne`和`enumSetting_ValueTwo`”。第五个参数（示例中为`null`）是一个可选的验证方法，它在`Enum`类型的“handles”中是不需要的。
>This creates a setting that requires the following strings to be provided in the language XML file:  
`enumSetting_title`, `enumSetting_desc`, `enumSetting_DefaultValue`, `enumSetting_ValueOne` and `enumSetting_ValueTwo`.
The fifth parameter (`null` in the example) is the optional validator method that is not required for `Enum` handles.

***

## The `SettingsChanged` event
当玩家改变了任意设置并关闭“Mod设置对话框”，`ModBase.SettingsChanged`方法就会被所有Mod请求调用。
重写该方法也许不是必要的，这取决于你的“设置”需要实现什么。然而，当需要的时候，请求调用“handles”和读取它们`Value`属性就够了，因为“handle”的值会被自动更新。
>When the player closes the Mod Settings dialog after changing any setting, the `ModBase.SettingsChanged` method is called for all mods.  
Depending on what your settings do, it may not even be necessary to override it, however. Requesting the handles and reading their `Value` property wherever it is needed may be enough, since the handle values are automatically updated.

***

## SettingsHandle reference
`SettingHandle`中有一些追加的**属性**，如果“handle”已经被创建，就可以明确地配置它们。
>These are additional properties in `SettingHandle` that allow for specific configuration after the handle has been created.

### Name
这是“设置”的独特代号，规定为`ModSettingsPack.GetHandle`方法的第一个参数。名称必须在创建该设置的Mod中是独一无二的，多个Mod可以有标识符相同的“设置”。
>This is the unique Id of the setting specified as the first argument to `ModSettingsPack.GetHandle`. The name must only be unique to the mod that created the setting, and multiple mods can have settings with matching identifiers.

### Validator
验证器方法可以在创建一个“handle”时，作为第五个参数传递给`ModSettingsPack.GetHandle`。它被用来确保玩家指定的和XML文件中储存的值有效。该方法接受一个`string`类型的参数，并且必须返回一个`bool`类型的值来指示提供的值是否有效。
第一次验证发生在“handle”被请求调用的时候。当验证器返回`false`时，就会放弃加载的值，并会把默认值指定给“handle”。
本类库为一些常见类型的“设置”提供了一些方便的验证器。下面的示例创建了一个只接受0至30之间整数的`int`类型“handle”：
>The validator method can be passed to `ModSettingsPack.GetHandle` as the fifth parameter when creating a handle. It is used to ensure that values assigned to the setting by the player and those loaded from the XML file are valid. The method receives a `string` argument and must return a `bool` to indicate that the provided value is valid.  
The first validation happens when the handle is requested. Retuning `false` in the validator during that phase will discard the loaded value and the default value will be assigned to the handle. The validator is also called when the player changes settings in the menu- rejecting the passed value will simply prevent it from being assigned to the handle.  
The library provides some convenience validators for commons setting types. This example creates an `int` type handle that will only accept values from 0 to 30 (inclusive):

```cs
Settings.GetHandle("intSpinner", "title", "desc", 0, Validators.IntRangeValidator(0, 30));
```

### VisibilityPredicate
这是一个可选的方法，当设置要在菜单中显示时就需要返回`true`。这对于创建那些只需要在开发者模式显示，或者和其他设置选项相关的“设置”是非常有用的。
>An optional method that should return `true` when the setting should be displayed in the menu. Useful for settings that should appear only in Dev mode or settings dependent on the values of other settings.

### CustomDrawer
这是个很棒的属性。一个允许Mod绘制它自己的“设置控件”，该控件会在“Mod设置”菜单中显示。它接受一个`Rect`类型的参数——屏幕中的绘制区域，并且必须返回一个`bool`类型的值来指示“设置”是否被更改。
这对于创建自定义数据类型的设置，或使按钮绑定一项操作是很有用的。
确保当“设置”改变时给`SettingHandle.Value`赋值，这会使它被正确的保存。
>This is a great one. A method that allows the mod to draw its own settings control to be displayed in the Mod Settings menu. It receives a `Rect` argument- the screen area to do the drawing in, and must return a `bool` to indicate if the setting has changed.  
This is useful for creating settings for custom data types or just making buttons to execute a certain action.  
Make sure to assign `SettingHandle.Value` when the setting changes so that it may be properly saved.

### NeverVisible
当该属性被设置为`true`，“handle”将不会在设置菜单中显示，并且不能被“重置所有设置”按钮重新设置。这对于储存独立于存档文件的自定义序列化数据结构是很有效的。
>When set to `true`, this handle will never appear in the settings menu and cannot be reset using the "Reset all" button. This is useful for storing custom serialized data structures that should be independent of save files.

### Unsaved
当该属性为`true`“handle”不会在XML文件中保存。有利于结合`CustomDrawer`属性在设置菜单中放置按钮。
>When `true`, this handle will not be saved to the XML file. Useful in conjunction with `CustomDrawer` for placing buttons in the settings menu.

### SpinnerIncrement
对`int`类型的“handles”很有帮助。指定“+”和“-”按钮会对“设置”的值产生多大改变。
>Useful for `int` handles. Specifies by how much the + and - buttons should change the setting value.

### CustomDrawerHeight
当使用`CustomDrawer`时，使用该属性有助于设定菜单中“设置”按钮的高度。默认的高度是32个像素。
>When using CustomDrawer this is useful to expand the height of the setting entry in the Menu. The default height is 32px.

### OnValueChanged
一个每当“设置”的值改变就被调用的委托。它可以替代需要重写的`SettingsChanged`方法。接受“handle”新的值作为它唯一的参数。
>A delegate that gets called each time the value of the setting changes. This is an alternative to overriding the `SettingsChanged` method. Receives the new value of the handle as its only parameter.

***

## Custom setting types
创建一个如“lists”或其他数据结构的复杂类型的“设置”是可行的。关键是要提供可以在这种数据结构和`string`类型之间互相转换的方法。可以通过继承`HugsLib.Settings.SettingHandleConvertible`类，然后使用新的数据类型作为“handle”的类型。
下面这个例子在一个“设置”中储存了一个序列化的`List<int>`：
>It is possible to create settings of complex types like lists and other data structures. The key is providing methods that will convert the data structure from and to its string representation. This is done by extending `HugsLib.Settings.SettingHandleConvertible` and using the new type as the type of a settings handle.  
This example stores a serialized `List<int>` in a setting:

```cs
public override void DefsLoaded() {
	var custom = Settings.GetHandle<CustomHandleType>("customType", "Label", null);
}
private class CustomHandleType : SettingHandleConvertible {
	public List<int> nums = new List<int>();

	public override void FromString(string settingValue) {
		nums = settingValue.Split('|').Select(int.Parse).ToList();
		Log.Message(nums.Join(","));
	}

	public override string ToString() {
		return nums != null ? nums.Join("|") : "";
	}
}
```

如果两个方法中出现了异常，控制台就会显示一个错误并且将“handle”的值重设为默认值。
>If an exception is raised in either method, an error is displayed in the console and the handle value is reset to its default.

为使自定义的数据类型变得有用，需要“handle”使用`CustomDrawer`来绘制特殊的控件。
或者，可以使用`NeverVisible`属性来创建一个隐藏“设置”来储存用户的某些数据。为了这个目的，可以使用一种特殊的调用——`HugsLibController.SettingsManager.SaveChanges();`。
但我们应该尽量少用它，因为磁盘的活动状态会被影响，并且其他Mod会传递和通知`ModBase.SettingsChanged`事件。
>For the custom data type to be useful, the handle should use `CustomDrawer` to draw a special control.  
Alternatively, `NeverVisible` could be used to create a hidden setting to store some kind of user data. For this purpose a special call can be made- `HugsLibController.SettingsManager.SaveChanges();`  
It's a good idea to use it sparingly, because disk activity is involved and all other mods are notified via the `ModBase.SettingsChanged` event.

推荐使用`null`作为自定义数据类型“设置”的默认值，以防止`Value`和`DefaultValue`引用同一个对象。
>It is recommended to use `null` as the default value for handles with custom data types to prevent both `Value` and `DefaultValue` from referencing the same object.

***

## Settings for non-library mods
没有创建一个继承自`ModBase`类的Mod的情况下，也能自定义“设置”。这通过调用`SettingsManager`类中的`ModSettingsPack`类型方法`GetModSettings`来实现：
>Mods that don't create a class extending `ModBase` can still make use of custom settings. This is done by requesting a `ModSettingsPack` from the settings manager:

```cs
HugsLibController.SettingsManager.GetModSettings("modIdentifier");
```

`modIdentifier`是Mod的一个独特标识符。该标识符会在XML文件中使用，所以应该避免使用空格和特殊字符。“manger”的返回值类型和`ModBase`继承类通过`Settings`属性存取的类型是一样的。
>`modIdentifier` being a unique identifier for the mod. The identifier will be used in the XML file, so avoid spaces and special characters. The type returned by the manager is the same one as a `ModBase` extending class would have access to through the `Settings` property.

***

## Optional settings (advanced)
有时候避免依赖HugsLib是更好的，而且当类库已经加载，配置“设置”依然是有效的。有一个取巧的方法可以实现它：在编译时引用类库，不过如果类库没有被加载，就要捕获运行时抛出的`TypeLoadException`。
>Sometimes it is preferable to avoid adding HugsLib as a dependency, but it would still he useful to have configurable settings when the library is loaded. There is a sneaky way to accomplish that: referencing the library at compile time, but catching the `TypeLoadException` that gets thrown at runtime if the library is not loaded.

牢记，所有引用自HugsLib的类必须包装成委托，和处理这项工作的`TypeLoadException`处理器一起放置在`try`代码块中。
>Keep in mind, that all references to HugsLib types must be made inside a delegate wrapped in a `try` block with a `TypeLoadException` handler for this to work.

这个例子创建了一个默认值为`true`的“设置开关”
代码和灵感来自Zenthar。
>This example creates a toggle setting with a default value of `true`.  
Code and idea contributed by Zenthar.
```cs
[StaticConstructorOnStartup]
public class TestMod {
	private static Func<bool> getSettingValue;

	static TestMod() {
		// execute once HugsLib is initialized
		LongEventHandler.QueueLongEvent(() => {
			InitializeSetting();
			Log.Message("Setting value: " + getSettingValue());
		}, null, true, null);
	}

	private static void InitializeSetting() {
		const bool defaultValue = true;
		// Need a wrapper method/lambda to be able to catch the TypeLoadException when HugsLib isn't present
		try {
			((Action) (() => {
				var settings = HugsLibController.Instance.Settings.GetModSettings("TestModIdentifier");
				// add a mod name to display in the Mods Settings menu
				settings.EntryName = "Test Mod";
				// handle can't be saved as a SettingHandle<> type; otherwise the compiler generated closure class will throw a typeloadexception
				object handle = settings.GetHandle("testSetting", "Setting label", "Setting description", defaultValue);
				getSettingValue = () => (SettingHandle<bool>) handle;
			}))();
			return;
		} catch (TypeLoadException) {
		}
		getSettingValue = () => defaultValue;
	}
}
```
