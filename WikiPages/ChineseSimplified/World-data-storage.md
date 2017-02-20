`UtilityWorldObject`是一个既方便又可靠的方法，使用存档文件储存和地图无关的Mod数据。
在A16之前，`MapComponent`曾经是用来接受事件和储存Mod数据的go-to方法。和激活地图同时发生，然而，他们不再可靠了——每个活跃的地图会创建自己组件的实例，而且当玩家放弃一个地图，该地图的所有组件都会丢失。
>`UtilityWorldObject`-s are a convenient and reliable way to store map-independent mod data within a save file.  
Before A16 `MapComponent`-s used to be the go-to method for receiving events and storing mod data. With the advent of simultaneously active maps, however, they are no longer reliable- every active map will create its own instance of a component and any components are lost when the player abandons a map.

详细地说，`UtilityWorldObject`——一个依附于世界地图的隐藏对象，可以通过游戏可靠地加载和保存。当从类库中请求调用一个UWO（缩写），和该类型匹配的现存对象就会被返回，否则就会创建一个新的并注入到世界。
为了使自定义的对象能储存他们的数据，`ExposeData`事件必须重写，而且照例，一些必要的字段要对`Scribe`方法公开。当游戏加载和保存时`ExposeData`事件就会被通知。
示例：
>Enter `UtilityWorldObject`- an invisible object attached to the world map, that is reliably loaded and saved with the game. When requesting a UWO from the library, either the existing object of the matching type is returned, or a new one is created and injected into the world.  
To allow the custom object to save its data, `ExposeData` must be overridden and the necessary fields exposed using `Scribe` methods, as usual. `ExposeData` will be called when the game is loaded and saved.  
Example:

```cs
public class StorageTest : ModBase {
	public override string ModIdentifier {
		get { return "StorageTest"; }
	}

	public override void WorldLoaded() {
		var obj = UtilityWorldObjectManager.GetUtilityWorldObject<WorldDataStore>();
		Logger.Message(obj.testInt.ToString());
		Logger.Message(obj.testString);
		obj.testInt++;
		obj.testString += "+";
	}

	private class WorldDataStore : UtilityWorldObject {
		public int testInt;
		public string testString;
			
		public override void PostAdd() {
			base.PostAdd();
			testInt = 1;
			testString = "+";
		}

		public override void ExposeData() {
			base.ExposeData();
			Scribe_Values.LookValue(ref testInt, "testInt", 0);
			Scribe_Values.LookValue(ref testString, "testString", "");
		}
	}
}
```
`WorldObject.PostAdd`是一个可选择重写的事件，在对象首次增加到世界时通知调用。
>`WorldObject.PostAdd` is an optional override and is called the first time the object is added to the world.

***

## `MapComponent extensions`
WorldObjects虽然不一定是最终的储存解决方案——`MapComponent`仍然对于储存哪些随`Map`存在和消失的数据是有用的。本类库增加了一些扩展方法来使它们变得更简单：
>WorldObjects, however, are not necessarily the ultimate storage solution- `MapComponent`-s are still useful for storing data that should live and die with a `Map`. The library adds some extension methods to make working with them easier:

### `MapComponentUtility.EnsureIsActive`
`MapComponent`的一个扩展方法，设定在其构造函数中调用。如果地图组件不存在，它就把组件注入地图。这对于允许Mod在已经储存的游戏中运行是有用的，因为创建地图时游戏还没有被激活。同时，Rimworld将会实例化一个`MapComponent`，但是它不会被马上通知调用，该方法讲使`MapComponent`接收到这一通知并且储存在地图中。
>A extension method for `MapComponent`, designed to be called in its constructor. It injects the map component into the map if it is not already present. This is useful for allowing a mod to work on saved games where it was not active at the moment of map creation. At the moment, Rimworld will instantiate a `MapComponent`, but it will not be added to the map if the map is being loaded from a save.  
Calling this method ensures that the `MapComponent` will receive calls and is saved with the map.

```cs
public class InjectedMapComponent : MapComponent {
	public InjectedMapComponent(Map map) : base(map) {
		this.EnsureIsActive();
	}
}
```

### `MapComponentUtility.GetMapComponent`
为`Map`提供的一个扩展方法，在`ModBase.MapLoaded`事件中是很有用的。它接受地图中提供类型的第一个`MapComponent`。
>An extension method for `Map` that is useful during the `ModBase.MapLoaded` event. It retrieves the first `MapComponent` of the provided type from the map.