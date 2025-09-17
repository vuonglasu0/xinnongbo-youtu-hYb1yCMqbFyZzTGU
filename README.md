> [【从UnityURP开始探索游戏渲染】](https://github.com)**专栏-直达**

# **自发光的基本原理**

$Cemissive=Memissive$

自发光是物体表面主动发射光线的现象，在光照模型中通常作为独立于外部光源的附加项。其核心特点是不受其他光照影响，但可以影响周围环境。

# **实现流程**

* ‌**定义发射颜色和强度**‌：确定基础发光颜色和亮度
* ‌**纹理采样 可选**‌：使用纹理控制发射图案
* ‌**HDR处理**‌：支持高于1.0的亮度值
* ‌**后期处理集成**‌：与Bloom等效果结合
* ‌**间接光照贡献 可选**‌：影响全局光照

# **Unity URP中的实现方案**

## **核心实现位置**

在URP中，自发光主要在以下文件中实现：

* `Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl`
* `Packages/com.unity.render-pipelines.universal/ShaderLibrary/SurfaceInput.hlsl`

## **关键代码实现**

* **unity\_emission\_shader**

  ```
  |  |  |
  | --- | --- |
  |  | half3 Emission(half3 emissionColor) |
  |  | { |
  |  | #if defined(_EMISSION) |
  |  | return emissionColor * _EmissionColor.rgb * _EmissionStrength; |
  |  | #else |
  |  | return 0; |
  |  | #endif |
  |  | } |
  ```
* Lighting.hlsl 部分示意

  ```
  |  |  |
  | --- | --- |
  |  | half3 CalculateFinalColor( |
  |  | InputData inputData, |
  |  | SurfaceData surfaceData) |
  |  | { |
  |  | // 基础光照计算 |
  |  | half3 color = ApplyLighting(inputData, surfaceData); |
  |  |  |
  |  | // 添加自发光 |
  |  | color += surfaceData.emission; |
  |  |  |
  |  | return color; |
  |  | } |
  ```

## **实现特点**

### ‌**材质属性配置**‌：

* `_EmissionColor`: 发光颜色(RGB)
* `_EmissionMap`: 发光纹理(可选)
* `_EmissionStrength`: 强度乘数

### ‌**HDR支持**‌：

* 通过FrameBuffer的HDR格式支持高亮度值
* 与Post-processing Stack的Bloom效果协同工作

### ‌**全局光照集成**‌：

* 通过Light Probe Proxy Volume影响动态物体
* 参与Reflection Probe的反射计算

### ‌**性能优化**‌：

* 使用`#if defined(_EMISSION)`编译分支
* 无发光材质自动跳过相关计算

## **URP选择此方案的原因**

### ‌**艺术家友好**‌：

* 直观的颜色和强度参数
* 纹理支持实现复杂发光图案

### ‌**物理合理性**‌：

* 正确的能量守恒处理
* HDR范围符合真实世界亮度

### ‌**性能平衡**‌：

* 轻量级实现不影响基础渲染性能
* 与URP的轻量级设计理念一致

### ‌**扩展性**‌：

* 容易与后期效果集成
* 支持自定义着色器变体

### ‌**跨平台一致性**‌：

* 在移动端和高端PC上表现一致
* 自动适配不同渲染管线配置

在URP中，自发光实现既保持了足够的表现力，又维持了轻量级的计算开销，特别适合移动平台和需要大量发光物体的应用场景。

---

> [【从UnityURP开始探索游戏渲染】](https://github.com):[wgetcloud全球加速器服务](https://wgetcloud6.org)**专栏-直达**

（欢迎*点赞留言*探讨，更多人加入进来能更加完善这个探索的过程，🙏）
