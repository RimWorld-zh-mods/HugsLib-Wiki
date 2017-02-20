本类库允许Mod创建一个简短的更新消息项目，来呈现给玩家。每个项目都与特定的Mod版本绑定，而且只会显示一次。
在地图加载后，不同Mod的消息项目在“New Mod Features”对话框中一同显示出来。[示例](http://i.imgur.com/0k6igKb.png)。
消息项目支持[Unity rich text](https://docs.unity3d.com/Manual/StyledText.html)，有一些基础的格式选项，还可以添加图片。
>The library allows mods to create short update news items to displayed to the player. Each item is tied to a specific version of the mod and is displayed only once.  
News items from different mods are displayed together in the "New Mod Features" dialog that appears after a map has been loaded. [Example](http://i.imgur.com/0k6igKb.png).  
News item content supports [Unity rich text](https://docs.unity3d.com/Manual/StyledText.html), has some basic formatting options and can contain images.

所有可获取的加载Mod消息项目，都可以通过“Mod设置”对话框中的按钮查看。
>All available news items for loaded mods can be viewed with a button in the Mod Settings dialog.

***

## Recommended usage
更新消息系统的作用，是允许Mod突出那些可能被玩家忽略的主要更新特性。
>The purpose of the update news system is to allow mods to highlight major new features that could otherwise go unnoticed by the player.

决定是否需要将你最近的更新突出在消息项目中的最简单方法，是从一般玩家的角度去考虑这些改变。
通常地，玩家只对那些他喜爱的Mod的重要增加项目感兴趣，而且一旦遇到大量的文字内容，他们很可能直接关闭窗口。
为了吸引玩家的注意力，最好使文本内容简洁明了，并且尽可能添加相应的图片。
>The easiest way to decide whether a recent change in your mod should be highlighted in a news item is considering the change from the perspective of the average player.  
Generally, only significant additions to their favorite mods are of interest to them, and they are likely to just hit the "Close" button if they run into a blob of text.
To best capture the attention of the player, it's a good idea to trim the text content down to a bare minimum and include relevant images, if possible.

***

## Requirements
为使消息项目正确的被识别和显示，必须满足以下两个条件：
* 你的Mod必须拥有一个`ModBase`的继承类，并且重写`ModIdentifier`属性。Mod的标识符必须与XML文件中的Def匹配。
* 你的程序集版本，必须匹配或超越消息项目的版本。如果你决定不增加程序集的版本，你可以使用`Version.xml`文件中的`overrideVersion`字段来指定Mod的当前版本，这在HugsLibChecker的readme文件中已经说明了。
>For a news item to be properly detected and displayed, two conditions must be satisfied:
>* Your mod must have a class that extends `ModBase` and overrides the `ModIdentifier` field. That mod identifier must match the one in the XML Def.
>* The version of your assembly must match or exceed the version specified in your news item. If you choose not to increment the version of your assembly, you can specify the current version of your mod using the `overrideVersion` field in the `Version.xml` file, as explained in the HugsLibChecker readme.

***

## Example
消息项目通过创建一个`UpdateFeatureDef`类型Def的XML文件来添加。
下面是一个通过创建`ModName/Defs/UpdateFeatureDefs/UpdateFeatures.xml`文件，添加了两个不同版本消息项目的例子：
>News items are added by creating an XML file that contains defs of the `UpdateFeatureDef` type.
Here's an example of a `ModName/Defs/UpdateFeatureDefs/UpdateFeatures.xml` file that creates news items for two different versions of a mod:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<Defs>
	<HugsLib.UpdateFeatureDef Abstract="true" Name="UpdateFeatureBase">
		<modNameReadable>Mod Name</modNameReadable>
		<modIdentifier>ModName</modIdentifier>
		<linkUrl>https://ludeon.com/forums/index.php?topic=17218</linkUrl>
	</HugsLib.UpdateFeatureDef>
	
	<HugsLib.UpdateFeatureDef ParentName="UpdateFeatureBase">
		<defName>ModName_1_1_0</defName>
		<assemblyVersion>1.1.0</assemblyVersion>
		<content>A basic news item with text content only.</content>
	</HugsLib.UpdateFeatureDef>
	
	<HugsLib.UpdateFeatureDef ParentName="UpdateFeatureBase">
		<defName>ModName_1_2_0</defName>
		<assemblyVersion>1.2.0</assemblyVersion>
		<content>First paragraph.|img:Things/Item/Meal/Fine,Things/Item/Meal/Lavish,Things/Item/Meal/Simple|caption:This is a caption.|Another paragraph with &lt;b&gt;rich text&lt;/b&gt;.</content>
	</HugsLib.UpdateFeatureDef>
</Defs>
```
当进入游戏，消息对话框看起来是[这样的](http://i.imgur.com/dGDhfVG.png)。
>When entering the game, the news dialog should look [something like this](http://i.imgur.com/dGDhfVG.png).

`defName`：消息Def仍然需要独特的名称来运行，尽管它们不会被系统使用。
`modNameReadable`：Mod的名称，它会在消息对话框中显示。
`linkUrl`：一个可选的论坛或创意工坊链接。
`assemblyVersion`：Mod的最低程序集版本，它将触发显示对应的消息项目。
`content`：消息项目的文本内容。可以添加额外的格式标记（如下）。既可以使用[HTML Entities](http://www.w3schools.com/html/html_entities.asp)添加大量的文本标签，编码标签；也可以使用[CDATA](https://www.w3.org/TR/REC-xml/#sec-cdata-sect)将整个文本封装。
>`defName`: The news Defs still require unique names to work, even though they are not used by the system.  
`modNameReadable`: The name of the mod, as it will be displayed in the news dialog.  
`linkUrl`: An optional link to your mod's forum or workshop page.  
`assemblyVersion`: The minimum version of your mod that will trigger this news item to be displayed.  
`content`: The text of your news item. Can contain additional formatting markers (see below). When adding rich text tags, encode tags using [HTML Entities](http://www.w3schools.com/html/html_entities.asp) or enclose the whole text with a [CDATA](https://www.w3.org/TR/REC-xml/#sec-cdata-sect).

***

## Formatting options
给消息项目增加格式标记，就可以插入图片和添加标题。这个系统是很简陋的，但是能将需要突出的要点和大量文本一起传递给玩家。
>Adding formatting markers to news item text allows to insert images and add captions for them. The system is fairly rudimentary, but in conjunction with rich text should be all that is needed to get the point across to the player.

### Pipe character (**|**)
“|”是用来创建新节点的分隔符——一个段落，一个图片的序列或者一个图片标题。
段落的示例：`One|Two|Three`
>The pipe is the delimiter that starts a new section- a paragraph, a sequence of images or an image caption.  
Paragraph example: `One|Two|Three`

### Image sequence (img:)
“|”字符后面紧接着`img:`和图片的名字，这样就在文本中可以插入一张图片。
可以是你Mod贴图文件夹中的路径，或者来自游戏本体。当使用你自己的图片，这个路径就和你的贴图文件夹相关。不需要包括文件的扩展名。
多个图片可以通过水平顺序来显示，只需要用逗号来分隔开它们的名字。
示例：`img:imgOne,imgTwo,imgThree`
>The pipe character immediately followed by `img:` and an image resource name will insert an image into the text.  
This can be an image from your own mod's Textures folder, or a texture from the base game. When using your own images, the path is relative to your Textures folder. The extension of the file should not be included.  
Several images can be displayed in a horizontal sequence by separating their name with a comma.  
Example: `img:imgOne,imgTwo,imgThree`

图片会全尺寸显示，并且仅在消息对话框打开时被游戏加载。这样Mod就可以添加那些只在消息项目中使用的图片，而且这些图片不会在游戏载入阶段加载或被添加到内存中。
>Images are displayed at full size and are loaded by the game only once the news dialog opens. This makes it possible for mods to include images that are used exclusively in news items without adding to the game load time or memory footprint.

### Image caption (caption:)
“|”字符后面紧接`caption:`，就可以在图片或图片序列的右边添加一个文本框。文本会垂直对准图片，并且文本框的高度会被图片的高度限制。
示例：`img:imageName|caption:This is a caption.`
>The pipe character immediately followed by `caption:` allows to append a block of text to the right side of an image or image sequence. The text will be aligned vertically to the image, and the height of the block will be limited by the height of the image.  
Example: `img:imageName|caption:This is a caption.`