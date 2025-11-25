# app-android
first work
媒体相册 Android 应用 - 详细设计文档
1.项目概述
本项目是一个基于 Kotlin 开发的原生 Android 媒体相册应用，完全采用传统 View 体系构建 UI。应用的核心目标是为用户提供一个简洁优雅的媒体管理工具，支持图片和视频的浏览、导入和管理功能。在 UI 设计上，项目参考了醒图等主流图片编辑应用的设计风格，注重视觉效果和用户体验的平衡。
应用采用 Material Design 设计语言，使用卡片式布局展示媒体内容。主界面通过 RecyclerView 实现网格布局的媒体库，配合 TabLayout 提供分类筛选功能。为了提升视觉吸引力，项目实现了两个自定义 View 组件，分别提供动态扫光效果和渐变裁切效果。这些自定义组件不仅满足了功能需求，也展示了 Android 原生绘图 API 的强大能力。
2.技术栈与依赖
本项目构建在 Android SDK 34 之上，最低兼容 Android 7.0（API 24），确保了广泛的设备覆盖率。开发语言选择 Kotlin 1.9，充分利用其简洁的语法和现代化的特性。项目使用 Gradle 7.x 作为构建工具，采用 Kotlin DSL 编写构建脚本，提高了配置的类型安全性和可维护性。
在第三方库的选择上，项目引入了 Glide 4.16.0 作为图片加载框架。Glide 不仅支持常规的 PNG 和 JPG 格式，还原生支持 WebP 和 GIF 动图，能够自动处理图片的解码、缓存和显示。对于视频播放功能，项目集成了 Google 官方的 Media3 ExoPlayer 1.2.0。ExoPlayer 相比传统的 MediaPlayer 提供了更灵活的扩展性和更好的性能表现，支持多种视频格式和流媒体协议。
UI 组件方面，除了 Android SDK 自带的 RecyclerView 和 ConstraintLayout，项目还使用了 Material Components 库来实现 TabLayout 和 FloatingActionButton 等现代化 UI 元素。这些组件不仅提供了美观的默认样式，还内置了符合 Material Design 规范的交互动画。
3.应用架构设计
应用采用简化的 MVC 架构模式，其中 Activity 充当控制器角色，负责协调 UI 和数据的交互。虽然没有使用 MVVM 或 MVP 等复杂架构，但通过合理的职责划分和模块化设计，代码依然保持了良好的可读性和可维护性。
MainActivity 作为应用的唯一主界面，承担了数据管理、用户交互处理和 UI 更新等核心职责。它持有两个独立的数据集合：sampleMediaItems 用于存储示例图片数据，galleryMediaItems 用于存储用户从相册导入的媒体。这种分离设计使得数据的来源和生命周期管理更加清晰。
MediaDetailActivity 专门负责全屏预览功能，根据传入的媒体类型动态切换显示组件。对于图片类型，使用自定义的 GradientClipImageView 展示；对于视频类型，则使用 ExoPlayer 的 PlayerView 组件。这种按需加载的设计既节省了内存资源，也优化了用户体验。
数据持久化采用 SharedPreferences 方案，虽然相比 Room 数据库功能较为简单，但对于本应用的数据量和复杂度而言完全够用。应用启动时会检查是否首次运行，首次运行时加载预设的示例数据，后续启动则从本地恢复用户的个性化内容。所有的数据修改操作（添加、删除）都会立即触发持久化保存，确保数据不会因应用异常退出而丢失。
4.自定义 View 实现详解
4.1 ShimmerImageView - 动态扫光效果
ShimmerImageView 是一个继承自 AppCompatImageView 的自定义组件，其核心功能是在图片上叠加一道从左上角到右下角移动的光晕效果。这个效果的实现完全基于 Android Canvas 和 LinearGradient API，不依赖任何第三方库。
扫光效果的绘制逻辑在 onDraw 方法中实现。首先计算视图的对角线长度，这个长度决定了光晕需要移动的总距离。使用勾股定理公式 sqrt(width² + height²) 可以精确得到这个值。接下来创建一个 LinearGradient 渐变对象，渐变方向是从左上到右下的对角线方向。渐变色由五个颜色节点组成：透明、半透明白色、完全白色、半透明白色、透明。这样的配置使得光晕呈现出中心明亮、边缘渐隐的自然效果。
动画部分使用 ValueAnimator 实现，动画值从 0 递增到对角线长度的 1.3 倍，确保光晕能够完全扫过整个图片区域。动画持续时间设置为 2 秒，采用线性插值器保证匀速运动。每当动画值更新时，会调用 invalidate() 触发重绘，从而产生连续的动画效果。为了避免性能问题，组件在 onSizeChanged 回调中重新初始化动画，确保视图尺寸变化后动画依然正确。一个重要的细节是 Paint 的混合模式设置。通过 setXfermode(PorterDuffXfermode(PorterDuff.Mode.SRC_ATOP))，扫光效果只会渲染在图片内容区域，不会溢出到透明背景上。这保证了视觉效果的专业性和一致性。
4.2 GradientClipImageView - 渐变裁切效果
GradientClipImageView 展示了 BitmapShader 的高级应用。这个组件不是简单地显示图片，而是对图片应用了两层渐变处理：底层使用 BitmapShader 绘制图片本身，上层叠加一个从上到下的渐变蒙版。
BitmapShader 的优势在于它能够将位图作为画笔的填充图案。在 onDraw 方法中，首先将 Drawable 转换为 Bitmap，然后基于这个 Bitmap 创建 BitmapShader。为了适配视图尺寸，需要计算适当的缩放比例。代码使用 max(width/bitmapWidth, height/bitmapHeight) 确保图片能够完全覆盖视图区域，这相当于 ImageView 的 CENTER_CROP 缩放模式。
渐变蒙版使用另一个 LinearGradient 实现，颜色从顶部的完全透明渐变到底部的半透明黑色。这种设计借鉴了现代图片应用的常见做法，既不遮挡图片主体内容，又能在底部区域为文字标题提供足够的对比度。两个 Paint 对象分别绘制图片和蒙版，通过 Canvas.drawRoundRect 方法为最终效果添加圆角，提升视觉精致度。
组件还实现了图片更新时的状态重置机制。当调用 setImageBitmap 或 setImageDrawable 时，会将 bitmapShader 置为 null 并触发重绘，确保新图片能够正确渲染。
4.3媒体管理功能实现
4.3.1权限管理与兼容性处理
Android 系统在不同版本上对存储权限的要求差异很大，本应用针对这一问题实现了完善的兼容方案。对于 Android 13（API 33）及以上版本，使用细粒度的 READ_MEDIA_IMAGES 和 READ_MEDIA_VIDEO 权限，这些权限更加安全且符合现代隐私保护理念。对于 Android 12 及更早版本，则使用传统的 READ_EXTERNAL_STORAGE 权限。
权限请求流程采用 ActivityResultContracts API，这是 Google 推荐的现代化方案，取代了已废弃的 requestPermissions 方法。用户点击添加媒体按钮时，应用首先检查当前权限状态，如果权限已授予则直接打开选择器，否则弹出系统权限请求对话框。整个流程通过 registerForActivityResult 机制实现，代码简洁且易于维护。
4.3.2媒体选择器实现
媒体选择功能同样采用了版本适配策略。在 Android 13 及以上设备上，使用 Google 提供的 Photo Picker API。这个 API 的优势在于它提供了系统级的统一界面，用户无需授予应用存储权限就能安全地选择照片和视频。Photo Picker 还支持多选、搜索等高级功能，用户体验优于传统文件选择器。
对于较老的 Android 版本，应用回退到使用 ACTION_GET_CONTENT Intent 方式。通过设置 MIME 类型为 “image/,video/”，用户可以从系统图库或文件管理器中选择媒体文件。虽然这种方式需要存储权限，但兼容性更好，能够在几乎所有 Android 设备上正常工作。
选中媒体后，应用会尝试获取 URI 的持久访问权限。这通过 takePersistableUriPermission 方法实现，允许应用在后续启动时继续访问用户选择的文件，而不需要重新请求权限。需要注意的是，Photo Picker 返回的某些 URI 不支持持久权限，因此代码使用 try-catch 块优雅地处理这种情况。
4.3.3 MIME 类型识别与分类
应用通过 ContentResolver.getType() 方法获取媒体文件的 MIME 类型，然后基于类型字符串进行分类。MIME 类型是一个标准化的字符串，格式为 “类型/子类型”，例如 “image/jpeg” 或 “video/mp4”。通过检查字符串的前缀，代码能够准确区分图片和视频。
对于图片类型，应用进一步细分为普通图片（IMAGE）、WebP 格式（WEBP）和 GIF 动图（GIF）。这种细分不仅有助于在 UI 上显示不同的标识符，也为后续可能的特殊处理（比如 GIF 的播放控制）预留了扩展空间。ContentResolver 还用于获取文件的显示名称，这个名称会作为媒体项的标题显示在界面上。
RecyclerView 与适配器设计
MediaAdapter 是连接数据层和视图层的关键组件。它继承自 RecyclerView.Adapter，负责将 MediaItem 数据对象转换为可视化的卡片视图。适配器采用 ViewHolder 模式，每个 MediaViewHolder 持有一个卡片视图的引用，包括 ShimmerImageView、标题 TextView 和类型指示器 TextView。
数据绑定逻辑在 bind 方法中实现。该方法接收一个 MediaItem 对象，根据其属性设置各个子视图的内容。类型指示器通过 when 表达式映射 MediaType 枚举到中文字符串。图片加载使用 Glide 库，一行代码就能处理从资源 ID 或 URI 加载、缩放裁剪、淡入动画等复杂操作。关键是 load(source) 方法接受多种类型的参数，适配器利用 Kotlin 的 Elvis 操作符优雅地处理两种数据源：item.uri ?: item.resourceId。
适配器还实现了长按删除功能。ViewHolder 的 itemView 设置了 OnLongClickListener，长按时会调用传入的回调函数，将待删除的 MediaItem 和其位置传递给 Activity。这种设计将删除逻辑的实现留给了上层，保持了适配器的职责单一性。扫光动画通过 imageView.setShimmerEnabled(true) 在每个卡片绑定时启用，确保所有媒体项都展现出一致的动态效果。
Tab 切换与过滤机制
应用主界面的 TabLayout 提供了三个标签页：全部、图片、视频。这个功能的实现基于一个简单但有效的过滤器模式。MainActivity 维护一个 currentFilter 变量，存储当前选中的过滤类型。当用户点击不同的标签时，OnTabSelectedListener 会捕获事件，更新 currentFilter 的值，然后重新加载并过滤数据。
过滤逻辑封装在 filterMediaItems 方法中。该方法接收完整的媒体列表，根据 currentFilter 的值返回过滤后的子集。对于”全部”选项，直接返回原列表。对于”图片”选项，使用 Kotlin 的 filter 扩展函数，只保留 type 为 IMAGE、WEBP 或 GIF 的项。对于”视频”选项，只保留 type 为 VIDEO 的项。这种函数式编程风格的代码简洁且易于理解。
每次过滤操作后，会调用 mediaAdapter.updateData(filteredItems) 更新界面。RecyclerView 的 notifyDataSetChanged 方法会触发视图的重新绑定，用户看到的就是过滤后的媒体列表。由于过滤是在内存中进行的，整个过程非常快速，用户几乎感受不到延迟。
7.数据持久化方案
应用使用 SharedPreferences 存储用户数据，这是 Android 提供的轻量级键值对存储机制。虽然不如 SQLite 或 Room 强大，但对于本应用的数据规模而言已经足够。数据以 JSON 格式序列化后保存为字符串，既方便人工调试，也保证了跨平台兼容性。
首次启动检测通过一个名为 KEY_FIRST_LAUNCH 的布尔值实现。应用启动时读取这个值，如果为 true（默认值），说明是首次运行，此时加载预设的示例数据并将标志位设为 false。这确保了用户第一次打开应用就能看到有内容的界面，而不是空白页面。后续启动时，直接从 SharedPreferences 恢复上次保存的数据。
保存操作分为三部分：示例项、相册项和 ID 计数器。示例项只保存 ID 列表，因为这些项的资源 ID 是固定的，可以根据 ID 在代码中重建。相册项则需要完整保存，包括 ID、类型、URI 字符串、标题和描述。GalleryMediaItem 类实现了 toJson 和 fromJson 方法，负责对象与 JSONObject 的互相转换。ID 计数器的保存确保了即使应用重启，新添加的媒体也不会与已有项产生 ID 冲突。
每次用户执行添加或删除操作时，saveData 方法会被立即调用。SharedPreferences 的 apply 方法是异步的，不会阻塞主线程，同时能保证数据最终一致性。这种设计在性能和数据安全之间取得了良好平衡。
8.详情页面实现
MediaDetailActivity 提供了媒体的全屏预览功能，支持图片和视频两种模式。Activity 的布局使用 ConstraintLayout 作为根容器，包含三个主要组件：GradientClipImageView 用于显示图片，PlayerView 用于播放视频，以及一个底部信息面板显示标题和描述。
页面初始化时，从 Intent 中提取传递的数据，包括媒体类型、资源 ID、标题、描述和可选的 URI 字符串。根据媒体类型，选择性地显示或隐藏对应的视图组件。对于图片类型，使用 Glide 将图片加载到 GradientClipImageView 中，自动应用渐变裁切效果。对于视频类型，创建 ExoPlayer 实例并配置播放参数。
ExoPlayer 的初始化过程包括几个关键步骤。首先通过 Builder 模式创建播放器实例，然后将其绑定到 PlayerView 上。根据数据来源（资源 ID 或 URI）构建 MediaItem 对象，这里使用了 Kotlin 的 when 表达式进行类型判断。资源文件需要转换为 “android.resource://包名/资源ID” 格式的 URI。设置 playWhenReady 为 true 使视频自动播放，repeatMode 设为 REPEAT_MODE_ALL 实现循环播放。
内存管理方面，Activity 重写了 onStop 和 onDestroy 方法，在适当的生命周期阶段释放 ExoPlayer 资源。这很重要，因为媒体播放器占用的系统资源较多，如果不及时释放会导致内存泄漏。
9.UI 设计与主题配置
应用的视觉风格遵循 Material Design 规范，使用浅色主题为主。状态栏颜色设置为白色，配合 windowLightStatusBar 属性使状态栏图标显示为深色，确保在白色背景上有足够的对比度。这种设计在现代 Android 应用中很常见，既美观又实用。
主色调选择蓝色（#2196F3），这是一个活泼但不刺眼的颜色，适合工具类应用。强调色使用粉色（#FF4081），在浮动按钮等需要引起注意的元素上使用。背景色采用浅灰色（#F5F5F5），与纯白的卡片形成微妙对比，增加界面层次感。
TabLayout 的样式通过 XML 属性配置，指示器颜色与主色调一致，选中标签文字也使用主色调，未选中标签使用次要文本颜色（#99000000）。这种设计使当前选中状态一目了然。卡片视图使用 MaterialCardView 实现，圆角半径设置为 12dp，投影高度 4dp，创造出浮在背景之上的视觉效果。
FloatingActionButton 固定在屏幕右下角，使用 24dp 的边距与边缘保持距离。按钮的图标是一个加号，直观地表达”添加”的语义。为了避免按钮遮挡最后一行的内容，RecyclerView 设置了 88dp 的底部内边距，同时启用 clipToPadding=“false” 允许内容滚动到内边距区域，实现优雅的滚动体验。
10.总结与展望
本项目成功实现了一个功能完整的媒体相册应用，涵盖了 Android 开发的多个重要领域：自定义 View 绘制、RecyclerView 列表展示、权限管理、文件选择、数据持久化、多媒体播放等。代码质量方面，通过合理的架构设计和职责分离，保持了较好的可维护性。
在性能优化上，应用采用了多项最佳实践。Glide 的三级缓存机制大幅减少了网络和磁盘 I/O。RecyclerView 的 ViewHolder 复用避免了频繁的视图创建。异步的 SharedPreferences 写入不会阻塞主线程。ExoPlayer 的及时释放防止了内存泄漏。这些细节的处理使应用能够流畅运行。
未来的改进方向可以考虑以下几个方面。首先是引入 ViewModel 和 LiveData，实现更规范的 MVVM 架构，提高代码的可测试性。其次可以添加图片编辑功能，利用 Android 的 Canvas API 实现滤镜、裁剪、文字添加等特性。再次是支持云端同步，让用户的媒体数据能够在多设备间共享。最后，可以考虑使用 Kotlin Coroutines 替代回调，使异步代码更加简洁优雅。
从技术学习的角度，这个项目充分展示了 Android 原生 View 体系的强大能力。即使不使用 Jetpack Compose 这样的现代化框架，通过对传统 API 的深入理解和灵活运用，同样能够构建出美观且功能丰富的应用。自定义 View 的实现过程尤其具有教育意义，它要求开发者深入理解 Canvas 绘图、Paint 配置、动画系统等底层机制，这些知识是任何高级 UI 框架的基础。
