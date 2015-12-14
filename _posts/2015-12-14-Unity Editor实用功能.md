不懂得偷懒的程序员不是好程序员  
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; --某某优秀程序员

写editor代码的目的：

- 工具化，解放劳动力；
- 美观，所见即所得。

##自定义菜单
使用 MenuItem 标签即可，例子如下。


```
public class AssetBundleEditor
{
    [MenuItem("Assets/Build AssetBundle From Selection - Track dependencies")]
    public static void BuildAssetBundle()
    {
        string path = EditorUtility.SaveFilePanel("Save Resource", "", "New Resource", "assetbundle");

        if (path.Length != 0)
        {
            // Build the resource file from the active selection.
            Object[] selection = Selection.GetFiltered(typeof(Object), SelectionMode.DeepAssets);
            AssetBundleTool.BuildAssetWithDependencies(selection, path);
        }
    }
}
```


##自定义 Component 面板

1. 使用CustomEditor；
2. 继承自 Editor；
3. 在 OnInspectorGUI 中进行自定义的绘制；
4. 将需要自定义Component 的组件设置为 private 或者设置 hideinhirarchy。

```
[CustomEditor(typeof(SceneSharedComponent))]
public class SceneSharedComponentEditor : Editor
{
    int choice = 0;
    public override void OnInspectorGUI()
    {
        SceneSharedComponent sharedCom = target as SceneSharedComponent;
        bool useLightMap = EditorGUILayout.Toggle("useLightMap", sharedCom.UseLightMap);
        sharedCom.UseLightMap = useLightMap;
        bool useQualityProp = EditorGUILayout.Toggle("useQualityProp", sharedCom.UseQualityProp);
        sharedCom.UseQualityProp = useQualityProp;
        if (useQualityProp)
        {
            string[] quality = new string[]{"low", "mid", "high"};
            choice = EditorGUILayout.Popup("Quality", choice, quality);
            if (choice == 0)
            {
                sharedCom.ObjQuality = Quality.QUALITY_LOW;
            }
            else if (choice == 1)
            {
                sharedCom.ObjQuality = Quality.QUALITY_MID;
            }
            else
            {
                sharedCom.ObjQuality = Quality.QUALITY_HIGH;
            }
        }
    }
}
```

##自定义窗口

1. 继承自 EditorWindow；
2. 使用 EditorWindow.GetWindow 进行显示；
3. 在 OnGUI 中进行 UI 处理。

```
public class NewExtendBehaviorWindow : EditorWindow
{
	private string path;
	
	public static void InitPath(string path)
	{
		NewExtendBehaviorWindow window = (NewExtendBehaviorWindow)EditorWindow.GetWindow (typeof(NewExtendBehaviorWindow));
		window.path = path;
		window.title = "NewBehavior";
		window.minSize = new Vector2 (100, 40);
	}	
	void OnGUI()
	{
		EditorGUILayout.BeginHorizontal();
		string hehaviorName = "NewExtendBehavior";
		hehaviorName = EditorGUILayout.TextField(hehaviorName);
		EditorGUILayout.EndHorizontal();
		if (GUILayout.Button("OK") && !string.IsNullOrEmpty(hehaviorName)) 
		{
			_CreateExtendBehavior(path, hehaviorName);
			Close ();
		}
	}
	private static void _CreateExtendBehavior(string path, string behaviorName)
	{
		string content = System.IO.File.ReadAllText(@Application.dataPath + "/Utils/ExtendBehaviorTemplate.cs");
		Debug.Log (@content);
		content = content.Replace("ExtendBehaviorTemplate", behaviorName);
		File.WriteAllText (path + "/" + behaviorName+ ".cs", content);
		AssetDatabase.Refresh ();
	}
}
```

##资源导入处理

1. 继承自 AssetPostprocessor；
2. 重写 OnPostprocessAllAssets；

```
public class DependenciesBy : AssetPostprocessor
{    
	private static void OnPostprocessAllAssets(string[] importedAssets, string[] deletedAssets,
	                                           string[] movedAssets, string[] movedFromAssetPaths)
	{
		Initialize ();
		for (int i=0; importedAssets != null && i < importedAssets.Length; ++i) 
		{
			repo.ImportAsset(importedAssets[i]);
		}
		for (int i=0; deletedAssets != null && i < deletedAssets.Length; ++i) 
		{
			repo.DeleteAsset(deletedAssets[i]);
		}
	}
}
```