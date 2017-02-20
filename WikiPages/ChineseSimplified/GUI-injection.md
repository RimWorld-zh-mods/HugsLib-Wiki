当给游戏添加改装功能，常常需要给现存的窗口添加一个或多个按钮。
通常，这需要绕道窗口的绘制方法，因为它对该问题有着自己的设置。
>When adding modded functionality to the game, it's often necessary to add a button or two to an existing window. Usually, this would require a detour of the window's drawing method, which comes with its own set of issues.

新的GUI注入系统移除了这些，并添加了一个简洁的途径来向游戏中的窗口注入代码。实际上，多个Mod可以同时项一个窗口注入内容，甚至可以完全替换原来的内容。
>The new GUI injection system does away with that and adds a clean way to inject code into *any* window in the game. In fact, multiple mods can add injections for the same window and even replace the window contents entirely.

## Basic usage
最简单的添加注入内容的途径，是使用`WindowInjection`属性。
该属性必须添加一个带`Window`和`Rect`类型参数的静态方法。那个`Window`类型的参数将使你的方法能被调用。
>The easiest way to add an injection is by using the `WindowInjection` attribute.
The attribute must be added to a static method with `Window` and `Rect` parameters. The argument of the attribute specifies the exact type of `Window` that will cause your method to be called.

下面的例子为"载入"对话框添加了一个按钮：
>This example adds a button to the "Load game" dialog:

```cs
[WindowInjection(typeof(Dialog_SaveFileList_Load))]
private static void DrawSaveDialogButton(Window window, Rect inRect) {
	var buttonSize = new Vector2(120f, 40f);
	if (Widgets.ButtonText(new Rect(0, inRect.height - buttonSize.y, buttonSize.x, buttonSize.y), "Hello")) {
		// do stuff				
	}
}
```
第一个参数是对绘制窗口的查询，而第二个则是窗口所包括的区域。
>The first parameter is a reference to the window being drawn, while the second is the content area of the window.

如果注入方法引起了一个异常，就会产生一个错误信息，并且会被移除以防止游戏性能被重复的错误填满。
>If the injection method causes an exception, it will produce an error message and be removed to keep the performance of the game from tanking due to repeated errors.

注意，注入内容并不一定要绘制GUI元素——它可以像钩子一样，用来链接执行其他代码。包括按照你的选择来关闭某个窗口，也可以打开一个新的窗口。
>Note, that an injection must not necessarily draw GUI elements- it could be used as a hook to execute other code, including closing that window and opening a new one of your choice.

## `InjectMode`
`WindowInjection`属性有一个可选的`Mode`类型的参数，该参数将指定注入内容该怎么运行。可选的值有`BeforeContents`，`AfterContents`和`ReplaceContents`。默认值是`AfterContents`，意味着你的代码将在一般的绘制代码之后运行。
>The `WindowInjection` attribute has an optional `Mode` parameter that allows to specify how the injection should execute. The possible values are `BeforeContents`, `AfterContents` and `ReplaceContents`. The default value is `AfterContents`, which means that your code will execute after the normal drawing code of the window.

`ReplaceContents`可以用来阻止通常的窗口内容被绘制，这对重制窗口的功能是很有用的。通常不建议这么做，因为它很容易破坏现有的功能，改变其他注入内容所依附的窗口。`ReplaceContents`不会阻止同一窗口的其他注入内容被调用。
>`ReplaceContents` can be used to prevent the usual window contents from being drawn, and can be useful if the intention is to completely rework the functionality of a window. This is generally not recommended, since it's easy to break existing functionality and alter a window that other injections rely on. `ReplaceContents` will not prevent other injections on the same window from being called.

## Replacing GUI elements
有时候改变现有按钮的功能是很重要的。尽管可以使用`ReplaceContents`接管窗口的所有内容，但这通常太过火了。
>Sometimes it is necessary to change what an existing button does. While it is possible to use `ReplaceContents` and take over the drawing of all the contents of that window, that is usually overkill. 

这有一个简单的实现方法——在现存的元素上绘制新元素。如果元素的尺寸和位置重合，就只会与后绘制的内容交互。
>There is an easier approach- just drawing the new element over the existing element. If the size and position of the elements match, only the element drawn last will be interactable.

标签同样可以用这种方法替换——可以在现存标签上绘制一个背景颜色的贴图，然后再在上面绘制新的标签。
>Labels can be replaced in the same way- a background-colored texture can be drawn on top of the existing label, and the new label can be drawn after that. 

```cs
var prevGuiColor = GUI.color;
GUI.color = new ColorInt(21, 25, 29).ToColor; // window background color
GUI.DrawTexture(targetRect, BaseContent.WhiteTex);
GUI.color = prevGuiColor;
```

## `ImmediateWindow`
`ImmediateWindow`是一个特殊的窗口实例，但是因为它继承自`Window`类，它一样可以简单地应用注入内容。
确保你绘制的是正确的窗口。如果不验证，注入内容就会绘入游戏中所有的当前窗口。
在反编译后的代码中可以找到数字ID，但必须添加一个负号来匹配正确的窗口。
>`ImmediateWindow` is a special case of window, but since it extends the `Window` class, it can have injections applied to it just as easily.  
The caveat is that all immediate windows have the same type, so it's necessary to check its ID field in your method to make sure you are drawing into the correct window. Without the check the injection would draw into all immediate windows in the game.  
The numeric ID can be found by looking at the relevant part of the decompiled source, but must come with a negative sign to properly match the window.

本示例在个人状态小窗口的右下角显示了“Hello”字符：
>This example writes "Hello" in the bottom right corner of the personal shield status mini-window:

```cs
[WindowInjection(typeof(ImmediateWindow))]
private static void DrawImmediateWindowLabel(Window window, Rect inRect) {
	var win = window as ImmediateWindow;
	if(window.ID != -984688) return;
	GenUI.SetLabelAlign(TextAnchor.LowerRight);
	Widgets.Label(new Rect(inRect.width-300, inRect.height-40, 300, 40), "Hello");
	GenUI.ResetLabelAlign();
}
```

使用这种验证窗口成员的方法，其他出于不同目的，而继承自同一类的不同窗口就也能被匹配。这适用于`Dialog_MessageBox`、`Dialog_NodeTree`等窗口。
>By the same logic of checking a window's members, other window types using the same type for different purposes can also be matched. This applies to `Dialog_MessageBox`, `Dialog_NodeTree`, and so on.

## Manual injection
如果你不喜欢这些属性，直接调用`WindowInjectionManager.AddInjection`也可以轻松添加注入内容。
这需要给注入内容提供一个独特的`injectionId`。像"namespace.typeName.methodName"这样，是一个好的命名方法，因为这可以在分析错误时，轻松定位拥有该注入内容的对象。
>If you're not fond of attributes, injections are also easy to add using a direct call to `WindowInjectionManager.AddInjection`.
This will require giving the injection a unique `injectionId`. Something like "namespace.typeName.methodName" is a good choice, since it allows to easier identify the owner of an injection when trying to diagnose issues.