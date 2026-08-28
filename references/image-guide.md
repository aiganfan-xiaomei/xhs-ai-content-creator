# 配图生成指南（参考生图）

## 核心原则
- 所有配图必须基于参考图生成（品牌IP/Logo/界面截图）
- 先选风格（见 [style-catalog.md](style-catalog.md) 的10种风格），再写Prompt
- 同一篇内容主风格1种+辅助风格≤1种
- 比例9:16竖屏，1152×2048，JPEG

## 风格选择
根据话题和文案类型自动推荐风格，也可让用户选择或组合：
- 教程型文案 → 白板手绘/照片涂鸦
- 种草型文案 → 真实摄影/3D卡通
- 避坑型文案 → 杂志编辑/纸片拼贴

完整10种风格详情见 [style-catalog.md](style-catalog.md)

## 内置参考图库（assets/，直接调用无需搜索）

Skill自带参考图，涉及以下品牌/角色时直接使用，省去搜索步骤：

### brand/ 品牌类
| 文件 | 内容 | 用途 |
|------|------|------|
| `assets/brand/doubao-avatar.png` | 豆包官方IP形象（3D卡通女生，深棕短发波波头，红围巾） | 豆包相关内容必用 |
| `assets/brand/doubao-work-logo.png` | 豆包工作App Logo（蓝色莫比乌斯环，白色圆角方形） | 豆包工作相关内容必用 |

### character/ 角色类
| 文件 | 内容 | 用途 |
|------|------|------|
| `assets/character/nina-turnaround.png` | 仁菜三视图（正面/侧面/背面，红发双马尾，海军蓝外套，红格裙） | 二次元风格、角色参考 |
| `assets/character/nijika-glasses.jpg` | 虹夏眼镜版（金发侧马尾，红框眼镜，日系插画） | 二次元风格、萌系角色 |
| `assets/character/anime-character-01.webp` | 二次元角色参考 | 通用二次元风格 |
| `assets/character/anime-character-02.jpeg` | 二次元角色参考 | 通用二次元风格 |

### scene/ 场景类
| 文件 | 内容 | 用途 |
|------|------|------|
| `assets/scene/bocchi-beach-wallpaper.png` | 孤独摇滚海滩壁纸（4人组，夏日海边，高饱和） | 二次元场景、夏日风格参考 |
| `assets/scene/bocchi-2024-01.jpeg` | 孤独摇滚2024场景图 | 二次元场景参考 |
| `assets/scene/bocchi-2024-02.jpeg` | 孤独摇滚2024场景图 | 二次元场景参考 |
| `assets/scene/bocchi-2024-03.jpeg` | 孤独摇滚2024场景图 | 二次元场景参考 |

**使用方式**：
- duck.ai：直接用 `bu.upload_file("input[type='file']", assets下的绝对路径)` 上传
- image_edit：用 `FileBatchUpload` 上传assets下的图片获取URL，再传入 `image_reference_url_list`
- 涉及豆包/豆包工作内容时，默认使用 brand/ 下的两张图作为参考
- 二次元/动漫风格内容时，可选用 character/ 或 scene/ 下的图作为风格参考

## 参考图采集（内置图库没有的才搜索）
1. 先检查内置 assets/ 是否有匹配的参考图
2. 没有的话：`image_search` 搜索 → `curl.exe -L -o` 下载到当天目录 `refs/` → `Read` 确认
3. 无特定品牌的话题跳过此步

## 方案A：duck.ai 参考生图（首选）

1. 读取 browser-use-automation skill，用 `computer_use_tool(plane="bu")`
2. `bu.navigate("https://duck.ai/")`
3. 上传参考图：`bu.upload_file("input[type='file']", 路径)`，多张逐张上传
4. 输入英文Prompt，必须说明：
   ```
   Using reference images: Image 1 is [形象]. Image 2 is [Logo].
   Style: [从style-catalog选的风格关键词].
   Generate a 9:16 vertical image: [场景描述]...
   ```
5. 等待20-30秒，base64保存：
   ```python
   result = bu.js("""const imgs=document.querySelectorAll('img');let l='';
   for(const i of imgs){if(i.naturalWidth>300&&i.src.startsWith('data:image/jpeg'))l=i.src;}return l;""")
   import base64; open(path,'wb').write(base64.b64decode(result.split(',',1)[1]))
   ```
6. 每张图生成后参考图被清空，下一张重新上传
7. 每日额度用完提示"已达到每日限额"→ 立即切方案B

## 方案B：内置 image_edit 参考生图（兜底）

1. `FileBatchUpload` 上传参考图获取URL
2. 调用 `image_edit`：
   ```python
   image_edit(request_list=[{
     "image_reference_url_list": ["URL1","URL2"],
     "prompt": "参考图1是[形象]，参考图2是[Logo]。风格：[风格名]。生成9:16竖屏图片：[描述]...4K超清",
     "width": 1152, "height": 2048
   }])
   ```
3. `curl.exe -L -o` 下载返回的URL
4. 可一次并行生成多张

## 每张图输出格式
```
图片N｜名称
风格：[风格编号+名称]
用途：[封面/功能/教程/对比/引导]
Prompt：[完整Prompt，含风格关键词]
图片文字：[后期添加的文字]
构图：[主体/文字/视觉重点位置]
```

## 图片数量与分配
- 每篇4-6张
- 图1封面（吸睛风格）
- 图2-3内容展示（清晰风格）
- 图4-5教程/对比/数据
- 图6关注引导（可爱/品牌风格）

## 统一视觉
- 年轻、可爱、科技感、轻松
- 可出现：饭盒、咖啡、奶茶、电脑、手机、AI机器人、像素元素
- 不每张都强行出现饭盒
- 风格统一但主题服务内容
