```
[MenuItem("Tools/BuildBundleWithTypeTree")]
public static void BuildBundleWithTypeTree()
{
	BuildPipeline.BuildAssetBundles(Application.streamingAssetsPath, BuildAssetBundleOptions.ChunkBasedCompression, BuildTarget.StandaloneOSX);
}

[MenuItem("Tools/BuildBundleWithoutTypeTree")]
public static void BuildBundleWithTypeTree()
{
	BuildPipeline.BuildAssetBundles(Application.streamingAssetsPath, BuildAssetBundleOptions.ChunkBasedCompression, BuildAssetBundleOptions.DisableWriteTypeTree, BuildTarget.StandaloneOSX);
}

[MenuItem("Tools/BuildBundleWithoutExtraNames")]
public static void BuildBundleWithTypeTree()
{
	BuildPipeline.BuildAssetBundles(Application.streamingAssetsPath, BuildAssetBundleOptions.ChunkBasedCompression, BuildAssetBundleOptions.DisableLoadAssetByFileName | BuildAssetBundleOptions.DisableLoadAssetByFileNameWithExtension, BuildTarget.StandaloneOSX);
}
```