## Detouring for fun and profit
重定向是将一个方法或属性的调用重新定向到另一个的方式。这使Mod可以改变游戏中本不能更改的功能。
**重定向会破坏原方法**，所以目标方法不得不重现原有的功能，然后才能加上想要添加的东西。
>Detouring is way to redirect all calls from one method or property to another. This allows mods to modify functionality in the base game that would otherwise not be modable.  
**Detouring destroys the original method**, so the destination method will have to reproduce its original functionality, in addition to whatever else it intends to do.

***

## <img align="left" src="http://i.imgur.com/6bfzmSv.png"/> Here be dragons
重定向是一个先进的改装特性，只作为最后手段来使用，如果没有其他合理的途径去实现想要的功能。可能有更好的解决方法——举个例子，一个Mod可以给“Thing”添加自定义的“Comp”，使用反射来操作私有的成员，通过继承来重写原功能并用该版本替换对象。在加载时修改Def也是一个选择，它可以在其他东西之间更改算法结构。
采用重定向方式有两个主要问题：
* 每个方法只能重定向一次。
这意味着，两个重定向同一方法的Mod会不兼容。如果尝试重定向一个已经重定向的方法，类库就会在控制台产生一个错误。这就容易在开发者模式下诊断错误，但是一般的玩家甚至不会看到错误。
* 更新你的Mod将需要额外的努力。
每次RimWorld的更新，Mod作者就必须浏览游戏基础代码中那些被重定向的方法。这就需要确保，在指定方法中复制的功能还和原来的匹配。
>Detouring is an advanced modding feature that is only to be used used as a last resort, when there is no other reasonable way to implement the functionality you are looking for. There may be better solutions- to name a few, a mod can add custom Comps to Things, use Reflection to access private members, and replace objects with versions that use inheritance to override original functionality. Modifying Defs at load time is also an option, which allows to alter ThinkTrees, among other things.
There are two main issues with taking the detouring approach:
>* Each method may only be detoured once.  
This means, that two mods that detour the same method will be incompatible with each other. The library will raise an error in the console when an attempt is made to detour an already detoured method. This may be easy to diagnose in Dev mode, but regular players would not even see the error.
>* Updating your mod will require additional effort.  
With each update to Rimworld, mod authors that use detours will have to look through each method they detoured in the base game. This is needed to make sure that the functionality they duplicated in the target method still matches that of the original.

***

如果你还在看，本类库提供了一些工具来使重定向更简单。注意，如果“Community Core Library”也被加载了，HugsLib就会转发所有的重定向请求给它，以提高兼容性。
>If you're still reading, the library provides some facilities to make detouring easier. Note, that if the Community Core Library is also loaded, HugsLib will forward any detouring requests to it for improved compatibility.

### Detour by attribute
分别用`DetourMethod`或`DetourProperty`修饰你的目标方法或属性，当声明这些的Mod被加载，类库就会自动应用合适的重定向。
>Decorating your target method or property with the `DetourMethod` and `DetourProperty` respectively will allow the library to apply the appropriate detours automatically when the declaring mod is being loaded.

### Detouring static methods
为了确保重定向正常工作，指定方法的参数总数，参数类型和返回值类型必须和原方法匹配。通常来说，较好的做法是在一个单独的静态类中声明你指定的方法，但这不是必须的。
示例重定向了`Widgets.ButtonText`，它在游戏中被用来绘制大部分的按钮，替换按钮的文本。注意，一个不同的方法而不是那个被重定向的方法，被用来处理实际的绘制工作。
>To ensure that a detour works properly, the argument count, argument types and the return type of the target method must match that of the original. It is not a requirement, but generally a good idea, to declare your target methods in a separate static class.  
This example detours `Widgets.ButtonText`, which is called to draw most buttons in the game, and replaces the button text. Note, that a different method than the one being detoured is used to do the actual drawing work.

```cs
[DetourMethod(typeof(Widgets), "ButtonText")]
public static bool ButtonText(Rect rect, string label, bool drawBackground, bool doMouseoverSound, bool active) {
	const string newLabel = "Test";
	return Widgets.ButtonText(rect, newLabel, drawBackground, doMouseoverSound, Widgets.NormalOptionColor, active);
}
```

### Detouring instance methods
示例方法接受一个额外的参数，这是对原方法中一个成员对象的引用。这相当于你在常规示例方法中使用的`this`关键字。为了接受这个参数，指定方法必须是静态的，并且声明为原方法父类型的扩展方法。所有增加的参数必须在引用类型参数之后。
下面的示例改变了说明面板中的字符串显示方法，面板在单位被选中时出现。
>Instance methods receive an additional argument, which is the reference to the object the original method is a member of. This corresponds to the `this` keyword you would use in a regular instance method. To receive this argument, the target method must be static and declared as an extension method of the original method's parent type. Any additional arguments must come **after** the self-referencing argument.
This example changes the string displayed in the description panel when any pawn is selected.

```cs
[DetourMethod(typeof(Pawn), "GetInspectString")]
private static string GetInspectString(this Pawn self) {
	return self.Label + " test";
}
```

### Detouring properties
重定向属性仅仅是重定向get，set方法至同一属性的语法糖。当重定向一个属性成员，操作该属性调用的对象经常是有用的。为了这个目的，我们可以创建一个继承类，它的基类需要包含这一属性。当在这个类的目标get或set中使用`this`关键字，就会引用重定向属性中调用的对象。不要在这个类中创建新的实例字段，并通过新的属性代码来操作他们，因为这会导致内存崩溃。
在下面的示例中，`Thing.HitPoints`属性被重定向了，这将阻止任何`Thing`的耐久值低于1，从而使东西不会损坏。这个例子从效率来看是浪费的，因为大量的反射调用。字段成员不是问题，因为它是静态的。
>Detouring a property is just syntactic sugar for detouring either the getter, setter or both property methods into an equivalent property. When detouring a member property, it is often useful to have access to the object the property vas invoked on. To that end we can create a class that extends the type containing the source property. When using the `this` keyword in the destination getter or setter within that class, it will refer to the object the detoured property was invoked on. **Do not** create new instance fields in that class to to access them from the new property code, as this will result in memory corruption.  
In this example the `Thing.HitPoints` property is detoured, preventing the hitpoints of any `Thing` from going below 1, thus making everything indestructible by damage. This example is inefficient performance-wise, because of the numerous reflection calls. The member field is not an issue because it is static.

```cs
public class Indestructible : Thing {
	public static FieldInfo hitPointsIntField = typeof(Thing).GetField("hitPointsInt", Helpers.AllBindingFlags); 
	[DetourProperty(typeof (Thing), "HitPoints", DetourProperty.Both)]
	public int _HitPoints {
		get { return (int)hitPointsIntField.GetValue(this); }
		set {
			if (value < 1) value = 1;
			hitPointsIntField.SetValue(this, value);
		}
	}
}
```

### Handling detour failures
当两个Mod重定向同一个方法或属性，只有首先加载的Mod会重定向成功。第二个Mod的重定向将引发一个错误。这就很难去调试，因为如果不是在开发者模式下，错误是在Log中悄悄显示的。
有一个方法可以消除你Mod重定向失败的可能性，向玩家展示一个错误对话框或者禁用Mod中的某一部分功能。2.2.0版本之后，可以使用`DetourFallback`属性来做到这一点。
该属性必须应用在有着特殊签名的静态方法中，这和你的目标方法或属性存在于同一个类中。
注意，在`ModBase`的所有子类实例化或初始化之前，重定向会被应用，回退处理程序会被调用。
示例在重定向因为已经被其他代码完成而失败的时候，向玩家显示一个错误对话框：
>When two mods detour the same method or property, the detour will only be applied for the one loaded first. The detour in the second mod will generate an error. This can be difficult to debug, since errors are silently written to the log when the game is not in Dev mode.  
It's a good idea to account for the possibility that your detour may not go through, and display an error dialog to the player or disable some part of your mod. Since version 2.2.0 there is a way to do that by using the `DetourFallback` attribute.  
The attribute must be applied to a static method with a specific signature, that exists within the same class as your destination method or property.  
Note, that detours are applied and fallback handlers are invoked before any `ModBase` descendants are instantiated or initialized.  
This example displays an error dialog box to the player when the detour fails due to being already taken:

```cs
public static class DetourFallbackTest {
	[DetourMethod(typeof(Pawn_RelationsTracker), "Notify_RescuedBy")]
	internal static void _Notify_RescuedBy(this Pawn_RelationsTracker t, Pawn rescuer) {
		// do usual detour stuff
	}

	[DetourFallback("_Notify_RescuedBy")]
	public static void DetourFallbackHandler(MemberInfo attemptedDestination, MethodInfo existingDestination, Exception detourException) {
		if (existingDestination != null) {
			LongEventHandler.QueueLongEvent(() => {
				Find.WindowStack.Add(new Dialog_MessageBox(string.Format("ModName: a required method was already detoured to {0}", existingDestination.FullName())));
			}, null, false, null);
		}
	}
}
```
`attemptedDestination`是重定向失败的目标方法或属性。这可以是`MethodInfo`或`PropertyInfo`。
`existingDestination`参数是对你已经重定向原方法的引用。如果不是因为重定向已完成而失败，该值就会是`null`。
`detourException`参数包含了尝试重定向而引发的异常。你可以使用`detourException.InnerException.Message`来查看细节。
>The `attemptedDestination` is the destination method or property of the attempted detour. This can be a `MethodInfo` or a `PropertyInfo`.  
The `existingDestination` parameter is a reference to the method your source method was already detoured to. Will be null if the failure is not due to the detour already being taken.  
The `detourException` parameter contains the exception that was generated during the detour attempt. You can use `detourException.InnerException.Message` to read the details.

可以给该属性提供多个应该接受失败信息的方法名称，或者一个也不提供——在这种情况下，该属性就要成为这个类中所有重定向的退路。
退路同样会处理其他重定向失败——大多明显情况是丢失原方法。这就在重定向方法位于其他Mod时很有用。
如果你手动应用重定向，你可以调用`DetourProvider.TryGetExistingDetourDestination`来查看你的原方法是否已经被重定向。
>The attribute can be provided with multiple method names it should accept failures from, or none at all- in which case is becomes the fallback for all detours in that class.  
Fallbacks will also handle other detouring failures- most notably a missing source method. This can be useful when detouring methods in other mods.  
If you are applying your detours manually, you can call `DetourProvider.TryGetExistingDetourDestination` to know if your source method has already been detoured.

### Classic detouring
也可以直接请求重定向，只需要提供原来的和目标的`MethodInfo`。
`DetourProvider.CompatibleDetour`在重定向失败时抛出一个异常，而`DetourProvider.TryCompatibleDetour`返回一个`bool`类型来指示操作是成功还是失败。
>A detour can also be requested directly, by providing the source and destination `MethodInfo`.
`DetourProvider.CompatibleDetour` will throw an exception if the detour fails, while `DetourProvider.TryCompatibleDetour` returns a `bool` to indicate the success or failure of the operation.

```cs
var source = typeof(Widgets).GetMethod("ButtonText", Helpers.AllBindingFlags, null, new[]{typeof(Rect), typeof(string), typeof(bool), typeof(bool), typeof(bool)}, null);
var destination = typeof(Detours).GetMethod("ButtonText", Helpers.AllBindingFlags);
DetourProvider.CompatibleDetour(source, destination);
```



