# Android 媒体相册应用（Media Gallery App）

一个基于 **Kotlin + Android 原生 View 体系** 的媒体相册项目，实现图片视频浏览、自定义特效、自定义组件、媒体导入、媒体分类等完整能力。

## 📌 项目特点（Features）

- 图片 & 视频浏览（支持本地导入）
- 自定义扫光效果 **ShimmerImageView**
- 自定义渐变裁切 **GradientClipImageView**
- 媒体分类筛选：全部 / 图片 / 视频
- 媒体类型自动识别（IMAGE / GIF / WebP / VIDEO）
- 使用系统 Photo Picker（Android 13+ 无需权限）
- SharedPreferences + JSON 持久化存储媒体数据
- 全屏预览（图片 / 视频）

## 🛠️ 技术栈（Tech Stack）

### **Language**
- Kotlin 1.9

### **Android**
- minSdk = 24  
- targetSdk = 34  
- Gradle Kotlin DSL 构建

### **Libraries**
| 功能 | 库 |
|------|------|
| 图片加载 | Glide 4.16.0 |
| 视频播放 | Google Media3 ExoPlayer 1.2.0 |
| UI | RecyclerView / ConstraintLayout / Material Components |
| 权限 | ActivityResultContracts |
| 媒体选择 | Photo Picker（Android 13+） |

## 🧱 项目架构（Architecture）

采用 **轻量级 MVC**：

- **MainActivity（控制器 + 媒体中心）**  
  - 媒体库  
  - 分类筛选  
  - 媒体导入  
  - 持久化存储  

- **MediaDetailActivity（全屏预览页面）**  
  - 图片：GradientClipImageView  
  - 视频：ExoPlayer  

- **Model**  
  - `MediaItem`：媒体对象定义（ID / type / uri / title 等）
## 🎨 自定义组件（Custom Views）

### 1. ShimmerImageView（动态扫光效果）

- 基于 Canvas + LinearGradient
- 对角线扫光动画（ValueAnimator）
- 使用 PorterDuffXfermode 让扫光只在图片区域绘制

### 2. GradientClipImageView（渐变裁切效果）

- 使用 BitmapShader 实现 CENTER_CROP 类似的铺满
- 叠加透明 → 半透明黑渐变蒙版
- 支持圆角裁切
## 📁 媒体管理逻辑

### ✓ 权限适配
- Android 13+：READ_MEDIA_IMAGES / READ_MEDIA_VIDEO  
- Android 12−：READ_EXTERNAL_STORAGE  
- 使用现代 API：ActivityResultContracts

### ✓ 媒体选择器
- Android 13+：Photo Picker  
- Android 12−：ACTION_GET_CONTENT  
- 兼容 URI 持久化（try-catch）

### ✓ MIME 类型识别
自动判断：
- IMAGE  
- GIF  
- WebP  
- VIDEO  
## 🖼️ RecyclerView 设计

- MaterialCard 风格（圆角 12dp、阴影 4dp）
- Glide 加载缩略图
- 类型标签（图片 / 视频）
- 长按删除
- Shimmer 动效


## 📺 全屏预览页面（Detail Page）

### 图片
- GradientClipImageView 显示
- 渐变裁切 + 圆角视觉更精致

### 视频
- ExoPlayer 播放器
- 自动循环、正确生命周期管理（onStop / onDestroy）

## 🎨 UI 设计风格

- Material Design 浅色主题
- 主色：`#2196F3`
- 强调色：`#FF4081`
- 背景：浅灰 `#F5F5F5`
- TabLayout + FAB + CardView


## 🚀 未来扩展方向（Future Work）

- 引入 MVVM（ViewModel + LiveData / Flow）
- 添加图片编辑能力（滤镜 / 裁剪 / 标注）
- 引入 Cloud 同步
- 升级 Jetpack Compose 版本
- 使用协程优化异步处理


## 📄 完整文档（Full Documentation）

➡ **完整详细技术文档请查看：**  
`Full_Document.md`

