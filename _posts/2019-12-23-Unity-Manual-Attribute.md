---
layout: post
title:  "Unity手册—Attribute汇总说明"
description: Unity脚本特性功能说明汇总
date:   2019-12-23 12:33:07 +0800
categories: [Unity]
tag: [参考手册,不定期更新]

---

内容取自官方API文档特性说明部分，用于开发参考，辅助开发提升开发效率；特性（Attribute）作用于脚本中的类、变量或方法的上，用[ ]包裹，用于表明其特殊行为，如 **[HideInInspector]**，使变量在 Inspector 面板隐藏；  

部分特性还未使用过，且网上资料很少，所以欠缺解释，后期补足

本文其他地址：[简书](https://www.jianshu.com/p/c6b897e6f2d6)	[知乎](https://zhuanlan.zhihu.com/p/99740444) 	[掘金](https://juejin.im/post/5e05a533e51d45581e441d31)

引用版本：[Unity 官方手册 ver. 2020.1](https://docs.unity3d.com/2020.1/Documentation/Manual/Attributes.html)	[UnityEngine API文档](https://docs.unity3d.com/2020.1/Documentation/ScriptReference/AddComponentMenu.html)	[UnityEditor API文档](https://docs.unity3d.com/2020.1/Documentation/ScriptReference/CallbackOrderAttribute.html)	[.NET特性文档](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/attributes/)  

## 常用特性

此处列出常用特性，功能说明在下方汇总

### 菜单 Menu

- AddComponentMenu
- MenuItem  
- CreateAssetMenuAttribute

### 监视器 Inspector

- HeaderAttribute
- HideInInspector  
- InspectorNameAttribute  
- SpaceAttribute  
- TextAreaAttribute  
- TooltipAttribute  
- ContextMenu  
- ContextMenuItemAttribute

### 对象/变量 Class/Property

- ColorUsageAttribute
- DisallowMultipleComponent
- MinAttribute  
- RangeAttribute
- RequireComponent  
- SerializeField
- Serializable
- NonSerialized

### 其他

- PostProcessBuildAttribute
- PostProcessSceneAttribute
- InitializeOnLoadAttribute
- InitializeOnLoadMethodAttribute  
- UnityAPICompatibilityVersionAttribute



## 官方文档特性汇总

### UnityEngine Attributes

- AddComponentMenu  

  参数：menuName 菜单名称

  作用于类上，将脚本置于 UnityEditor 顶部菜单栏 Component 菜单中的任意选项，而不局限于 Component->Scripts 选项下

  将脚本置于 UnityEditor 顶部菜单栏 Component 菜单中的任意选项，而不局限于 Component->Scripts 选项下

  ```c#
  [AddComponentMenu("WarlGComponent/WarlGAttribute")]
  public class WarlGAttributeSample : MonoBehaviour
  {
  }
  ```

  ![](https://nzjltg.dm.files.1drv.com/y4mCjPs4fEHFZB_sfq3qeLfxHpzlfdMfGZmlQ5M49HhL8R5w0laslecL6jrAWpVKpp9MkdAY-ca0goJxleF9buePsL9uQ8BX3ZKFgsNujvIJQeh_YU0saVzGK6tMvUlAMhCYcXl8hV2OnnCZFjRzzUoi8YNTE42h_ks3jRx6uCWPTDGCTrVjFB7d6yRCae8k0FKOMKf1ypDw65PbHcSfK9hXA?width=660&height=410&cropmode=none)

  ![](https://pgeczw.dm.files.1drv.com/y4mk2FGokVFq7YKEKUKpshZ_LWVTtJ_D33zfuWkrI3ed9ATTZ_4N_UfXXM2_wrT16nlExewx3lsaqNPr89dbFD7URACnBHYvjC-yKoMvUOuZ5wYIc6ceeBMNk8s0LgEz4vWnMFDv2eqmfaGLB4HRNl7wMRv9abqhL9c6yq8KDewLigvNkcupQxDm7bhJmCgc8FmnsTUjPNwGjnjScwmEk4QDg?width=256&height=252&cropmode=none)

- AssemblyIsEditorAssembly  

  作用于程序集，具有此属性的程序集中的任何类将被视为编辑器类

- BeforeRenderOrderAttribute  

  参数： order 调用顺序

  作用于方法上，被作用的方法会被注册为 [Application.onBeforeRender](https://docs.unity3d.com/2020.1/Documentation/ScriptReference/Application-onBeforeRender.html) 的事件回调，以给定顺序从低到高顺序调用

- ColorUsageAttribute  

  参数：showAlpha 是否显示 Alpha 配置项、HDR 是否高动态范围

  作用于 Color 类型变量上，使其可在取色面板配置显示 Alpha 和采用 HDR 标准

  ```c#
  [ColorUsage(true,true)] 
  public Color colorpicker;
  ```

  <img src="https://nzivzq.dm.files.1drv.com/y4mlpn1C1LmoR7lklDzzvBmVx8Ye03JwIGZTEYicW-sMgbHoUj08yvI8hH-SUYVHXKNgYyIc2cSa2AR6gPkP59yhJulHdpCAyBIMbC9PYxFJT8n548PiQLAhkIBFWNEtCSOL17ZAU9AQzz8FH5dUauGtu0fLFqGdPTY-FVZHMekdinP0V6zntJYUNN0YXpxpJzv71Y3iEjQdTDg1yIfuNttgw?width=660&amp;height=533&amp;cropmode=none" style="zoom:50%;" />

- ContextMenu  

  参数：itemName 菜单项名称

  作用于非静态方法上，在脚本 Inspector 的 context menu 上添加额外操作（Inspector 上脚本右击菜单）

  ```c#
  [ContextMenu("WarlG Context Menu")]
  void CustomContext()
  {
    Debug.Log("WarlGContext");
  }
  ```

  

  <img src="https://nzjebw.dm.files.1drv.com/y4mViTidyeyqQWRylAf6yatn7aPugMUOZ7uSoD-teXkZzW-Z2CYlKU14Veo9uwRq5D4LGwcXEl2ajcSjqF3K_D0YPRgSBvJ1D9dsNY7aW6t5vERR1aIY_BgA8miTEwUix1pd_pDQS1y-CGSrNB71qglJbj8-JpGiChNlW2bvo6TlU-boJfv2DbQyn9SxiKWICRMepa-IoNi_BFekJK2tfuvgQ?width=445&amp;height=660&amp;cropmode=none" style="zoom:50%;" />

- ContextMenuItemAttribute  

  参数：name 菜单名、function 方法名

  作用于变量上，给作用的变量在Inspector上添加右键菜单方法

  ```c#
  [ContextMenuItem("ResetString", "ResetSampleString")]
  public string SampleString = "";
  void ResetSampleString()
  {
    	SampleString = "WarlGSampleString";
  }
  ```

  <img src="https://nzj6wa.dm.files.1drv.com/y4m0dYeTXRFbtE7UgPzB4wrr8jRu98iQakucGkRruonX7UpTlXvz1oGGzey7L2ETir2LsCHPAfmwSv95gPNLsoqpM5Inqr1PCExO3Cz944S1WMZ83N0d3W8W6UFfO3LBsjIulhGW1SWV6QfCpYRS3IvWRFCkpakmXToz2Kot8m2WvNlrsEacsfLoeMOXBPL5kVTFaPT98L7jql16q5ASAEwEg?width=453&amp;height=660&amp;cropmode=none" style="zoom:50%;" />

- CreateAssetMenuAttribute  

  参数：fileName 文件名、menuName 菜单名、order 顺序  

  作用于可脚本化的类上，自动列入Assets/Create 的子菜单中，添加创建".asset"文件的快捷路径

  ```c#
  [CreateAssetMenu(fileName = "WarlGAssetSample.asset", menuName = "WarlGAssetMenu/WarlGAssetItem")]
  public class WarlGAssetMenuSample : ScriptableObject
  {
      public string assetName = "WarlGAssetSample";
      
  }
  ```

  <img src="https://nzkwqq.dm.files.1drv.com/y4mcoE3RsmdOTQ59WgSOzNsIhtc33uP-db-Uo-1vyLQtEPwL-Qb4LthRkxza289GEBRY1iH9udEExIj59AgxpyQOqoqudfLiNW7M-XbFNUTwkgiutnWy5PYkq3mYLodUINLqbSGEkNIzg-jm5pXBATcjy8EzCbJE5UoILWUPeUNAM30rQUrrZrf6R2mNJAYHBIBdnkx-fLXpkMT2LgyXjM1Ng?width=660&amp;height=475&amp;cropmode=none" style="zoom:67%;" />

  <img src="https://nzlyna.dm.files.1drv.com/y4mLFP8HGHYLFdnufznOMi2Ie4mN6d55tZ6iNnSn6QdtMC1H97tEiZ6d9H7xR19hC_DFeWdx49B_J3j5F_V-Nb7locQjCyiw5Sn8MbK2edC1hQrAuI0gccQu51OaFS2HR90W5uWfPmOMES1O7TCqL1JN5wMqYSZsM_bM5z3GSzpzaIIUUnqYbemnNLjtLWaWPWj8Uz6Ka3UW6EomLjt0mdG8w?width=660&amp;height=284&amp;cropmode=none" style="zoom:67%;" />

- CustomGridBrushAttribute  

  参数：defaultBrush 是否替代默认笔刷、defaultName 笔刷名、hideAssetInstances隐藏该笔刷于 tile palette 的资源实例、hideDefaultInstance 隐藏默认笔刷实例  

  作用于类上，将其定义为 grid brush，可在 palette window 中使用  

  ```c#
  [CustomGridBrushAttribute(true,true,true,"WarlGBrush")]
  public class WarlGBrush : GridBrushBase
  {
  }
  ```

  <img src="https://nziohg.dm.files.1drv.com/y4mq-Z3AGhbqXVWMWD0BBfroCfVSDKIJT0Y5fPrgCGnpSOYLz0XTnvhND30LxtXYsGWOLPL51BGS32MkECDhf24uS7syQAnzbtIKE0571KXYiwkZrhLb7ns9ZNgWhEV2HTgfHdX1IZPWrTDHGCvDOSPTMDBLOAOUsDvp74MfEQzSUIHwFlMAdslhLWQjRSyv7Zzj478go6yB9puydeRUaZ5cw?width=559&amp;height=660&amp;cropmode=none" style="zoom:67%;" />

- DelayedAttribute

  作用于基本数据类型 float, int 和 string 上，使其在Inspector被修改时不会立即生效，而是在按下回车或移除焦点时

- DisallowMultipleComponent  

  作用于类上，禁止 GameObject 添加多个相同类型组件

- ExcludeFromObjectFactoryAttribute  

  作用于类上，禁止该类及其子类被 [ObjectFactory](https://docs.unity3d.com/2020.1/Documentation/ScriptReference/ObjectFactory.html) 的方法创建  

- ExcludeFromPresetAttribute  

  作用于类上，禁止该类实例被用于创建 [Preset](https://docs.unity3d.com/2020.1/Documentation/ScriptReference/Presets.Preset.html) 对象

- ExecuteAlways  

  作用于类上，使实例对象脚本无论在 Edit Mode、 Play Mode 都执行；注意区分 Edit Mode 和 Play Mode 代码执行逻辑，Edit Mode 下 Update 仅在 Scene 更改时执行，OnGUI 仅在接收到"仅限Editor之外事件"时（如：EventType.ScrollWheel）且不响应Editor快捷键，OnRenderObject 和其他渲染回调方法在场景重绘时调用。

- ExecuteInEditMode  

  作用于类上，使脚本实例在 Edit Mode 下执行，同ExecuteAlways

- GradientUsageAttribute  

  参数：hdr 是否使用 HDR

  作用于 Gradient 类型变量上，配置是否支持 HDR

- GUITargetAttribute  

  作用于方法上（不强制），控制 OnGUI 代码在哪一 Display 展示（Game 窗口下可切换 Display）

- HeaderAttribute  

  参数：header 名称

  作用于变量上，给变量在 Inspector 中添加标题头  

  ```c#
  [Header("WarlGHeader")] 
  public string header;
  ```

  <img src="https://nzjzeq.dm.files.1drv.com/y4mJUt6z1RAbOsQgXS_WXzt9Wdp5kbN-9mBozaDGqc9ct5keBGEixtt5gfoXbHLeFRJh3PPknEml2ZyGQkk50_IO5gL72Ls5ZvHjw0Ghkp-U78QPirsiFtMzgUULSnlr-M_dVgTToLVp5q9eQ8bqbdnc65sIhOQgE1t0S0WBZWvdcb1RlLsDZ66hfBtAziS4yzABkETifRf6gnbKBDS6u79yg?width=660&amp;height=394&amp;cropmode=none" style="zoom:67%;" />

- HelpURLAttribute  

  参数：URL 文档链接

  作用于类上，为类添加说明链接，可在 Inspector 脚本组件上的"?"标志跳转

  ```c#
  [HelpURL("https://warl-g.github.io/")]
  public class WarlGAttributeSample : MonoBehaviour
  {
  }
  ```

- HideInInspector  

  作用于变量上，使变量在 Inspector 不可见，但可被序列化

- ImageEffectAfterScale  

  作用于 Image Effect （效果待验证），使 Image Effect 在动态分辨率阶段之后渲染

- ImageEffectAllowedInSceneView  

  作用于 Image Effect （效果待验证），使 Image Effect 渲染在 Scene view 相机上

- ImageEffectOpaque  

  作用于 Image Effect （效待验证果），使 Image Effect 渲染在透明图形前、不透明图像后，使扩展了深度缓冲（depth buffer）如环境光遮蔽（SSAO）等效果的 Image Effect 仅作用于不透明像素；此属性可用于减少具有后期处理（post processing）的场景中的视觉伪像（visual artifacts）数量

- ImageEffectTransformsToLDR  

  在使用 HDR 渲染条件下，渲染 Image Effect 切换到 LDR 渲染

- ImageEffectUsesCommandBuffer  

  使用命令缓冲区实现图像效果时使用该属性，Unity会将场景渲染到RenderTexture中，而不是实际的目标中

- InspectorNameAttribute  

  参数：displayName 显示名称

  作用于枚举值，更改枚举值在 Inspector 上显示的名称

  ```c#
  public enum WarlGEnum
  {
    WarlGEnum1 = 0,
    [InspectorName("WarlGEnumAnotherName2")]
    WarlGEnum2 = 1,
    [InspectorName("WarlGEnumAnotherName3")]
    WarlGEnum3 = 2,
  }
  ```

  ![](https://pgg3qw.dm.files.1drv.com/y4mCVGjZwJMQLnSapspcVEoaSpUr8FbjHfGVAwrwU0N_bJMAVLc7hCiDf07i_a4vKnmxRIknE47IQzjTLtGOaSeYlxUwbvbRLADuW5ek2gD_VAioa6hFr1vHv-vY5gsZRbDQsxZJbY7V5JdtHeWlDyNLQc8s7BO3W1rW_dyBAjt4f0v3u0_SIMge9veAXkjcSOdd9i8Drv82DNPIKd7BlGE4g?width=458&height=285&cropmode=none)

- MinAttribute  

  变量：min 最小值

  作用于 int 和 float 约束变量最小值

  ```c#
  [Min(10)]
  public int minValue;
  ```

- MultilineAttribute  

  作用于 string ，使 string 在 Inspector 显示多行文本区

- PreferBinarySerialization  

  作用于 ScriptableObject 派生类，使用二进制序列化取代项目资源序列化，可提升读写效率，提高压缩表现且无法直接识别，但当资源文件包含多个资源时无法使特定资源使用二进制序列化，嵌入场景的组员也会无视二进制序列化

- PropertyAttribute  

  自定义特性的基类，用于创建自定义特性派生类

- RangeAttribute  

  作用于 int 和 float 变量，限制范围值，在 Inspector 上显示范围条  

  ```c#
  [Range(2, 7)] 
  public int rangValue;
  ```

  ![](https://nzlf4w.dm.files.1drv.com/y4m5fpu3LVdVzWe4TsTtwICoY5iR4hPXT0yDpMg8Tpo3NDk-h_IOQco6Ot5STQfvmRdukRiqk5aOb2eCpFdRDRzEIgA65F3McFQo-2RjOHv5e6xaN-jtM7DxxC08PDFDkz80z30_zTnAJ9q4kIaz7oQ3Yv1r_gs6JWqt-ZS9JFAaQ4azNleRco4-Prnv3aByOk4rnx7ea_Kpv6KOd3N5Pt59g?width=457&height=281&cropmode=none)

- RequireComponent  

  参数：requiredComponent 组件类型

  作用于类，  在挂载该脚本同时会自动挂载该脚本依赖的组件，且删除时弹出警告

  ```c#
  [RequireComponent(typeof(SpriteRenderer))]
  public class WarlGAttributeSample : MonoBehaviour
  {
  }
  ```

- RPC  

  已废弃

- RuntimeInitializeOnLoadMethodAttribute  

  作用于静态方法，允许运行时情况下加载游戏后，无需用户行为即可初始化运行时类方法；被该特性标记的方法会在游戏加载完成 Awake 方法执行后执行，但所有被标记方法执行顺序不定

- SelectionBaseAttribute  

  作用于类，使被挂载的 GameObject 优先被选中；如当前 GameObject 为子物体，当点击该物体时会默认选中父物体，在该特性脚本挂载到子物体后会优先选中子物体

- SerializeField  

  作用于所有类型，被标记的私有类、变量等会被强制序列化；

  注该序列化为 Unity 内部的序列化系统与 .NET 序列化方法无关，Unity 序列化系统可操作非静态公共字段和被 SerializeField 标记的非静态私有字段，无法序列化属性和静态字段；有关序列化内容可见[链接](https://docs.unity3d.com/2020.1/Documentation/Manual/script-Serialization.html)

  ```c#
  [SerializeField]
  private int privateValue;
  ```

- SerializeReference  

  作用于引用对象，使 Unity 序列化对象为引用类型；Unity 默认除 UnityEngine.Object 派生类以外类型都序列化为值类型

- SharedBetweenAnimatorsAttribute  

  作用于类，使特定 StateMachineBehaviour 仅实例化一次病在所有 Animator 之间共享

- SpaceAttribute  

  作用于所有字段，在 Inspector 添加空行像素

- TextAreaAttribute  

  参数：maxLines 最大行、minLines 最小行

  作用于 string 类型，在 Inspector 添加高度灵活可滑动的文本区

- TooltipAttribute  

  作用于字段，为字段在 Inspector 添加悬浮解释文字

  ```c#
   [Tooltip("This is a Tooltip")]
   public int toolTip = 0;
  ```

  <img src="https://nzkppg.dm.files.1drv.com/y4mMi4MIDVj1FSq-5u04drKASyMD59ZUfzeOkjyZmcrrYuWdAYR9jAI4OIQk47VG-Ia2Zvw2KW0E3wJCzHF0dr2SZXHoKz-nCjvesx9EuWMtDaTKetIntbLB1OADqibk8X9I9P-RYwXAiV0utqgZjFKxbJmxwGVYwryZH__phiMyo5rDOSAG3kx0W5VhE0FielPRGdA-bUNOc1S5Hu53SehUQ?width=660&amp;height=414&amp;cropmode=none" style="zoom:50%;" />

- UnityAPICompatibilityVersionAttribute  

  作用于程序集，声明程序集兼容版本

### UnityEngine.Animations

- NotKeyableAttribute  

  作用于属性，使其不可动画（无法添加关键帧）

### UnityEngine.Scripting  

- AlwaysLinkAssemblyAttribute  

  无论程序集是否被引用都强制 UnityLinker 处理程序集

- PreserveAttribute  

  作用于所有类型，避免类、方法、字段或属性在字节码精简时被删除

### UnityEngine.Serialization

- FormerlySerializedAsAttribute  

  作用于字段，在不丢失已序列化值得情况下重命名一个字段  

### UnityEngine.TestTools  

- ExcludeFromCoverageAttribute  

  将程序集、类、构造方法、方法、结构体从代码覆盖率方法中剔除  

### UnityEngine.VFX  

- VFXEventAttribute  

  作用于类，用于处理发送到使用 [VisualEffect.SendEvent](https://docs.unity3d.com/2020.1/Documentation/ScriptReference/VFX.VisualEffect.SendEvent.html) 系统的属性   

### UnityEditor Attributes

- CallbackOrderAttribute  

  需要回调的特性基类

- CanEditMultipleObjects  

  作用于类，使自定义编辑器支持多对象编辑

- CustomEditor  

  作用于类，声明该类为运行时自定义编辑器类

- CustomEditorForRenderPipelineAttribute  

  作用于类，声明该类为基于渲染管线创建的运行时自定义编辑器类

- CustomPreviewAttribute  

  作用于字段，为指定类型在 Inspector 添加额外预览

- CustomPropertyDrawer  

  声明自定义 PropertyDrawer 用于哪种运行时序列化类或属性

- DrawGizmo  

  作用于静态方法，使任意组件支持 gizmo 渲染方法

  ```c#
  [DrawGizmo(GizmoType.Selected | GizmoType.Active)]
  static void DrawGizmoForMyScript(MyScript scr, GizmoType gizmoType)
  {
    Vector3 position = scr.transform.position;
    if (Vector3.Distance(position, Camera.current.transform.position) > 10f)
      Gizmos.DrawIcon(position, "MyScript Gizmo.tiff");
  }
  ```

- FilePathAttribute  

  用于指定文件对项目目录或引用目录的相对地址

- InitializeOnEnterPlayModeAttribute  

  作用于静态方法，使编辑器类方法在进入 Play Mode 时初始化  

  ```c#
  static int s_MySimpleValue = 0;
  
  [InitializeOnEnterPlayMode]
  static void OnEnterPlaymodeInEditor(EnterPlayModeOptions options)
  {
    Debug.Log("Entering PlayMode");
    if (options.HasFlag(EnterPlayModeOptions.DisableDomainReload))
      s_MySimpleValue = 0;
  }
  ```

- InitializeOnLoadAttribute  

  作用于静态方法（构造方法），当 Unity 加载时或脚本重编译时初始化编辑器类

- InitializeOnLoadMethodAttribute  

  作用于静态方法，Unity 加载时执行编辑器类方法

- LightingExplorerExtensionAttribute  

  作用于类，标记 Lighting Explorer 的扩展类，支持每条渲染管线一个扩展

- MenuItem  

  作用于静态方法，为主菜单和 Inspector context menu 添加选项

  可添加快捷键：%（ctrl 或 command，区别于 Windows 或 MacOS）	#（Shift）	&（Alt）

  \#LEFT （左Shift）LEFT, RIGHT, UP, DOWN, F1 .. F12, HOME, END, PGUP, PGDN等

  如：`WarlGMenu/Item #&g` 即 shift-alt-G		`WarlGMenu/Item _g` 即仅适用 `G` 键  

  ```c#
  [MenuItem("WarlGMenu/Item #&g")]
  static void WarlGMenuItem()
  {
  }
  ```

- PreferenceItem  

  已废弃

- SettingsProviderAttribute  

  作用于静态方法，注册新的 SettingProvider

- SettingsProviderGroupAttribute  

  作用于静态方法，注册多个 SettingProvider

- ShaderIncludePathAttribute  

  已废弃

### UnityEditor.Callbacks

- OnOpenAssetAttribute  

  作用于静态方法，当 Unity 将要打开一个 Asset 时，该方法被调用需满足如下条件：

  `static bool OnOpenAsset(int instanceID, int line)`  
  `static bool OnOpenAsset(int instanceID, int line, int column)`    

  返回 true 表明由方法处理该 Asset，返回 false 表明外部打开  

- PostProcessBuildAttribute  

  作用于静态方法，构建 Player 之后被调用

  ```c#
  [PostProcessBuildAttribute(1)]
  public static void OnPostprocessBuild(BuildTarget target, string pathToBuiltProject) {
    Debug.Log( pathToBuiltProject );
  }
  ```

- PostProcessSceneAttribute  

  作用于静态方法，构建 Scene 或 Play Mode 调用 SceneManager.LoadScene 方法后被调用  

### UnityEditor.EditorTools  

- EditorToolAttribute  

  将 EditorTool 注册为全局工具或一特地类型自定义编辑器  

### UnityEditor.Experimental.AssetImporters  

- CollectImportedDependenciesAttribute  

  作用于静态方法，指定哪些方法声明对引入的 Asset 有依赖

  ```c#
  [CollectImportedDependencies(typeof(ModelImporter), 1)]
  public static string[] CollectImportedDependenciesForModelImporter(string assetPath)
  {
    if (assetPath.Equals(s_DependentPath))
      return new[] { s_DependencyPath };
    return null;
  }
  ```

- ScriptedImporterAttribute  

  作用于类，注册一个  ScriptedImporter 的派生类为自定义 Asset Importerl   

### UnityEditor.Localization.Editor  

- LocalizationAttribute  

  本地化程序集  

### UnityEditor.Rendering  

- ScriptableRenderPipelineExtensionAttribute  

  将条件应用在类查询过滤器上，过滤器类型取决于当前使用的 ScriptableRenderPipeline  

### UnityEditor.ShortcutManagement

- ClutchShortcutAttribute  

  将静态类注册为一个 clutch shortcut  

- ShortcutAttribute  

  将静态类注册为一个 action shortcut  

- ShortcutBaseAttribute  

  ShortcutAttribute 和 ClutchShortcutAttribute 的抽象基类  

### UnityEditor.UIElements  

- UxmlNamespacePrefixAttribute  

  在程序集中为名称空间定义XML命名空间前缀   

### Unity.Burst  

- BurstDiscardAttribute  

  作用于方法和属性，将其从原生代码编译中移除  

### Unity.Collections.LowLevel.Unsafe

- DeallocateOnJobCompletionAttribute  

  官网暂无详细解释，直译任务结束析构

- NativeDisableParallelForRestrictionAttribute  

  官网暂无详细解释，直译限制并行

- NativeFixedLengthAttribute  

  容器空间固定不变

- ReadOnlyAttribute  

  标记结构体成员为只读

- WriteOnlyAttribute  

  标记结构体成员为只写  

### Unity.Jobs.LowLevel.Unsafe  

- JobProducerTypeAttribute  

  所有任务接口类型都需要被 JobProducerType 标记，用于被 Burst ASM Inspector 编译为执行方法

### .NET

- Serializable  

  作用于类或结构体，表名其可被序列化；注 仅非抽象、非通用自定义类可被序列化

- NonSerialized  

  声明变量不可被序列化

- 其他暂不列