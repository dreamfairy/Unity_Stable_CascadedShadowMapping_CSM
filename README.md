# Unity_Stable_CascadedShadowMapping_CSM
Stable and unflick CascadedShadowMapping
稳定，无抖动CSM

原po主未解决阴影平移和旋转的阴影边缘抖动，闪烁问题， 本仓库在原po主基础上修改
原po主仓库地址:https://github.com/chenyong2github/CascadedShadowMapping

原po效果


![](https://github.com/dreamfairy/Unity_Stable_CascadedShadowMapping_CSM/blob/main/Screenshots/w.gif?raw=true)


修正后效果


![](https://github.com/dreamfairy/Unity_Stable_CascadedShadowMapping_CSM/blob/main/Screenshots/Stable/1.gif?raw=true)
![](https://github.com/dreamfairy/Unity_Stable_CascadedShadowMapping_CSM/blob/main/Screenshots/Stable/2.gif?raw=true)


修改区域

1.平移抖动解决：从阴影相机本地空间转换到世界空间前，要在本地空间计算  平面宽度->阴影贴图纹理 映射， 目的是阴影相机每次位移时都保证为对应的阴影像素整数倍，防止出现半像素情况
```
float unitPerPixel = maxDis / depthTextures[k].width;

Vector3 pos = lightCamera_Splits_fcs[k].nearCorners[0] + (lightCamera_Splits_fcs[k].nearCorners[2] - lightCamera_Splits_fcs[k].nearCorners[0]) * 0.5f;

pos.x = Mathf.Floor(pos.x / unitPerPixel) * unitPerPixel;
pos.y = Mathf.Floor(pos.y / unitPerPixel) * unitPerPixel;

dirLightCameraSplits[k].transform.position = dirLightCameraSplits[k].transform.TransformPoint(pos);
```

2.缩放及旋转解决：锁定对角线距离，每次计算使用最大对角线并缓存，下次不再计算
```
float crossDis =
    Vector3.Magnitude(lightCamera_Splits_fcs[k].farCorners[2] - lightCamera_Splits_fcs[k].nearCorners[0]);

float maxDis = Mathf.Max(wDis, hDis);
maxDis = crossDis;

if (!crossDisCached.TryGetValue(k, out maxDis))
{
    if (crossDis != 0)
    {
        crossDisCached.Add(k, crossDis);
        maxDis = crossDis;
    }
}
else
{
    if (crossDis > maxDis)
    {
        crossDisCached[k] = crossDis;
    }
}
```