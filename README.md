# LuaAppX 插件开发教程

## 概述

LuaAppX 插件系统允许开发者扩展编辑器功能，提供自定义的工具和界面。本教程将指导您从零开始创建功能丰富的插件。

## 目录

- [插件基础结构](#插件基础结构)
- [插件配置文件](#插件配置文件)
- [事件系统](#事件系统)
- [用户界面设计](#用户界面设计)
- [实用工具集成](#实用工具集成)
- [代码模板系统](#代码模板系统)
- [高级功能](#高级功能)
- [实战示例](#实战示例)
- [发布与调试](#发布与调试)

## 插件基础结构

### 标准目录结构

```
插件名称/
├── init.lua              # 插件配置文件
└── config/
    └── events/
        ├── EditorActivity.lua    # 编辑器事件处理
        └── DataTableConfig.json  # 数据配置(可选)
```

### 最小插件示例

**init.lua**:
```lua
appname="我的插件"
packagename="com.example.myplugin(插件包名)"
mode="plugin"
appcode="1.0"
author="您的名字"
describe="插件描述"
supported2={
  LuaAppX2={mincode=1099,targetcode=9999}
}
```

**config/events/EditorActivity.lua**:
```lua
{
  onCreate = function (savedInstanceState)
    -- 插件初始化代码
    print("插件已加载")
  end
}
```

## 插件配置文件

### init.lua 配置项详解

| 配置项 | 类型 | 必需 | 说明 |
|--------|------|------|------|
| appname | string | ✓ | 插件显示名称 |
| packagename | string | ✓ | 插件包名(唯一标识) |
| mode | string | ✓ | 固定为"plugin" |
| appcode | string | ✓ | 插件版本号 |
| author | string | - | 作者信息 |
| describe | string | - | 插件描述 |
| supported2 | table | ✓ | 兼容性配置 |

### 版本兼容配置

```lua
supported2={
  LuaAppX2={
    mincode=1099,    -- 最低支持版本
    targetcode=9999  -- 最高支持版本
  }
}
```

## 事件系统

### 可用事件类型

#### 1. onCreate 事件
插件初始化时调用，用于设置功能按钮和初始化资源。

```lua
{
  onCreate = function (savedInstanceState)
    local FunctionUtil = require "activities.editor.FunctionUtil"
    
    -- 添加功能按钮
    FunctionUtil.add("按钮名称", function()
      -- 按钮点击事件
    end)
  end
}
```

#### 2. onCreateOptionsMenu 事件
当编辑器创建选项菜单时调用。

```lua
{
  onCreateOptionsMenu = function(menu)
    menu.add("菜单项")
    .onMenuItemClick = function(v)
      -- 菜单点击事件
    end
  end
}
```

### 核心工具类

```lua
-- 编辑器功能工具
local FunctionUtil = require "activities.editor.FunctionUtil"

-- 编辑器工具
local EditorUtil = require "activities.editor.EditorUtil"

-- 编辑视图工具
local EditView = require "activities.editor.EditView"

-- 活动工具
local ActivityUtil = require "utils.ActivityUtil"

-- 获取插件路径
local PluginPath = getPluginPath("您的包名")
```

## 用户界面设计

### 1. 基础对话框

```lua
-- 简单提示对话框
function showSimpleDialog(title, message)
  local MaterialAlertDialogBuilder = luajava.bindClass"com.google.android.material.dialog.MaterialAlertDialogBuilder"
  
  MaterialAlertDialogBuilder(activity)
    .setTitle(title)
    .setMessage(message)
    .setPositiveButton("确定", nil)
    .show()
end
```

### 2. 自定义布局对话框

```lua
function showCustomDialog()
  local layout = {
    LinearLayout,
    orientation="vertical",
    padding="16dp",
    {
      TextView,
      text="标题",
      textSize="18sp",
      textStyle="bold",
      layout_marginBottom="10dp"
    },
    {
      EditText,
      id="inputText",
      hint="请输入内容",
      layout_width="match_parent"
    },
    {
      Button,
      id="okButton",
      text="确定",
      layout_width="match_parent",
      layout_marginTop="10dp"
    }
  }
  
  local dialog = AlertDialog.Builder(activity)
    .setView(loadlayout(layout))
    .create()
  
  dialog.show()
  
  -- 绑定事件
  local okButton = dialog.findViewById(R.id.okButton)
  okButton.onClick = function()
    local input = dialog.findViewById(R.id.inputText)
    local text = tostring(input.getText())
    -- 处理输入的文本
    dialog.dismiss()
  end
end
```

### 3. 底部弹出对话框

```lua
function showBottomSheetDialog()
  local BottomSheetDialog = luajava.bindClass"com.google.android.material.bottomsheet.BottomSheetDialog"
  local BottomSheetBehavior = luajava.bindClass"com.google.android.material.bottomsheet.BottomSheetBehavior"
  
  local dialog_view = loadlayout({
    LinearLayout,
    layout_width = "fill",
    layout_height = 0.8 * activity.getHeight(),
    orientation = "vertical",
    {
      TextView,
      text = "底部对话框",
      textSize = "20sp",
      gravity = "center",
      layout_height = "wrap"
    }
  })
  
  local bottomDialog = BottomSheetDialog(activity)
  bottomDialog.setContentView(dialog_view)
  
  -- 设置透明背景
  dialog_view.getParent().setBackgroundResource(android.R.color.transparent)
  
  -- 设置默认高度
  BottomSheetBehavior.from(dialog_view.getParent()).setPeekHeight(0.8 * activity.getHeight())
  
  bottomDialog.show()
end
```

## 实用工具集成

### 1. 编辑器操作

```lua
-- 获取当前编辑器文本
local function getEditorText()
  local editor = activity.findViewById(android.R.id.edit)
  if editor then
    return tostring(editor.getText())
  end
  return ""
end

-- 设置编辑器文本
local function setEditorText(text)
  local editor = activity.findViewById(android.R.id.edit)
  if editor then
    editor.setText(text)
  end
end

-- 在光标位置插入文本
local function insertText(text)
  editor.pasteText(text)
end

-- 格式化代码
local function formatCode()
  EditView.format()
end
```

### 2. 文件操作

```lua
-- 保存当前文件
FunctionUtil.add("保存", function()
  EditorUtil.save()
end)

-- 新建文件
FunctionUtil.add("新建", function()
  FunctionUtil.OpenGreateDialog()
end)

-- 获取当前文件路径
local function getCurrentFilePath()
  return FunctionUtil.getLuaPath()
end
```

### 3. 项目操作

```lua
-- 打包项目
FunctionUtil.add("打包", function()
  ActivityUtil.new("build", {luaproject})
end)

-- 查看日志
FunctionUtil.add("日志", function()
  ActivityUtil.new("logs")
end)

-- 备份项目
FunctionUtil.add("备份", function()
  local ProgressMaterialAlertDialog = require "dialogs.ProgressMaterialAlertDialog"
  local wait_dialog = ProgressMaterialAlertDialog(activity).show()
  
  activity.newTask(function(path, MyToast, res)
    local FileUtil = require "utils.FileUtil"
    local result = FileUtil.backup(path)
    MyToast(result and (res.string.backup_succeeded .. ": " .. result) or res.string.backup_failed)
  end, function()
    wait_dialog.dismiss()
  end).execute({luaproject, MyToast, res})
end)
```

## 代码模板系统

### 1. JSON 配置文件结构

创建 `config/events/DataTableConfig.json`:

```json
[
  {
    "title": "分类名称",
    "data": {
      "subtitle": ["模板1", "模板2", "模板3"],
      "content": ["代码内容1", "代码内容2", "代码内容3"]
    }
  }
]
```

### 2. 读取和使用模板

```lua
-- 读取 JSON 配置
local function loadTemplates()
  local PluginPath = getPluginPath("您的包名")
  local DataTablePath = string.format("%s/config/events/DataTableConfig.json", PluginPath)
  local DataLuaTable = require("cjson").decode(require("rawio").iotsread(DataTablePath))
  return DataLuaTable
end

-- 显示模板选择器
local function showTemplateSelector()
  local templates = loadTemplates()
  
  -- 使用 ExpandableListView 显示分类模板
  local adp = ArrayExpandableListAdapter(activity)
  
  for i = 1, #templates do
    adp.add(templates[i].title, templates[i].data.subtitle)
  end
  
  -- 处理模板选择
  ExpandableListView.onChildClick = function(l, view, pid, cid)
    local selectedTemplate = templates[pid + 1].data.content[cid + 1]
    editor.pasteText(selectedTemplate)
    EditView.format()
    dialog.dismiss()
  end
end
```

### 3. 常用模板示例

```json
[
  {
    "title": "UI控件",
    "data": {
      "subtitle": ["按钮", "文本框", "列表"],
      "content": [
        "{\n  MaterialButton,\n  text='按钮',\n  onClick=function()\n    -- 点击事件\n  end\n}",
        "{\n  EditText,\n  hint='请输入',\n  id='editText'\n}",
        "{\n  ListView,\n  id='listView',\n  layout_width='fill',\n  layout_height='fill'\n}"
      ]
    }
  }
]
```

## 高级功能

### 1. 网络请求

```lua
-- 异步 HTTP 请求
function httpRequest(url, callback)
  Http.get(url, nil, nil, nil, function(code, content, cookie, header)
    if code == 200 and content then
      callback(true, content)
    else
      callback(false, "请求失败")
    end
  end)
end

-- 使用示例
httpRequest("https://api.example.com/data", function(success, data)
  if success then
    print("数据:", data)
  else
    print("错误:", data)
  end
end)
```

### 2. 文件操作

```lua
-- 读取文件
function readFile(path)
  local file = io.open(path, "r")
  if file then
    local content = file:read("*all")
    file:close()
    return content
  end
  return nil
end

-- 写入文件
function writeFile(path, content)
  local file = io.open(path, "w")
  if file then
    file:write(content)
    file:close()
    return true
  end
  return false
end
```

### 3. 数据持久化

```lua
-- 保存插件配置
function saveConfig(key, value)
  local PluginPath = getPluginPath("您的包名")
  local configPath = PluginPath .. "/config.json"
  
  local config = {}
  local file = io.open(configPath, "r")
  if file then
    local content = file:read("*all")
    file:close()
    config = require("cjson").decode(content) or {}
  end
  
  config[key] = value
  
  file = io.open(configPath, "w")
  if file then
    file:write(require("cjson").encode(config))
    file:close()
  end
end

-- 读取插件配置
function loadConfig(key, defaultValue)
  local PluginPath = getPluginPath("您的包名")
  local configPath = PluginPath .. "/config.json"
  
  local file = io.open(configPath, "r")
  if file then
    local content = file:read("*all")
    file:close()
    local config = require("cjson").decode(content)
    return config[key] or defaultValue
  end
  
  return defaultValue
end
```

## 实战示例

### 完整插件示例：代码生成器

**init.lua**:
```lua
appname="代码生成器"
packagename="com.example.codegen"
mode="plugin"
appcode="1.0"
author="开发者"
describe="快速生成常用代码"
supported2={
  LuaAppX2={mincode=1099,targetcode=9999}
}
```

**config/events/EditorActivity.lua**:
```lua
{
  onCreate = function (savedInstanceState)
    local FunctionUtil = require "activities.editor.FunctionUtil"
    local EditorUtil = require "activities.editor.EditorUtil"
    local EditView = require "activities.editor.EditView"
    
    -- 添加代码生成按钮
    FunctionUtil.add("生成代码", function()
      showCodeGenerator()
    end)
    
    -- 代码生成器界面
    function showCodeGenerator()
      local layout = {
        LinearLayout,
        orientation="vertical",
        padding="16dp",
        {
          TextView,
          text="代码生成器",
          textSize="20sp",
          textStyle="bold",
          gravity="center",
          layout_marginBottom="20dp"
        },
        {
          Button,
          text="生成Activity布局",
          layout_width="match_parent",
          layout_marginBottom="10dp",
          onClick=function()
            generateActivityLayout()
          end
        },
        {
          Button,
          text="生成ListView适配器",
          layout_width="match_parent",
          layout_marginBottom="10dp",
          onClick=function()
            generateListViewAdapter()
          end
        },
        {
          Button,
          text="生成HTTP请求",
          layout_width="match_parent",
          layout_marginBottom="10dp",
          onClick=function()
            generateHttpRequest()
          end
        }
      }
      
      local dialog = AlertDialog.Builder(activity)
        .setView(loadlayout(layout))
        .create()
      
      dialog.show()
    end
    
    -- 生成Activity布局
    function generateActivityLayout()
      local code = [[
layout = {
  LinearLayout,
  orientation="vertical",
  layout_width="fill",
  layout_height="fill",
  background="#ffffff",
  {
    MaterialToolbar,
    id="toolbar",
    layout_width="fill",
    layout_height="56dp",
    title="标题"
  },
  {
    NestedScrollView,
    layout_width="fill",
    layout_height="fill",
    {
      LinearLayout,
      orientation="vertical",
      layout_width="fill",
      layout_height="wrap",
      padding="16dp",
      -- 在这里添加内容
    }
  }
}

activity.setContentView(loadlayout(layout))
]]
      editor.pasteText(code)
      EditView.format()
    end
    
    -- 生成ListView适配器
    function generateListViewAdapter()
      local code = [[
-- 数据
local data = {
  {title="项目1", content="内容1"},
  {title="项目2", content="内容2"},
  {title="项目3", content="内容3"}
}

-- 列表项布局
local item = {
  LinearLayout,
  orientation="vertical",
  layout_width="fill",
  layout_height="wrap",
  padding="16dp",
  {
    TextView,
    id="title",
    layout_width="fill",
    layout_height="wrap",
    textSize="16sp",
    textColor="#333333"
  },
  {
    TextView,
    id="content",
    layout_width="fill",
    layout_height="wrap",
    textSize="14sp",
    textColor="#666666",
    layout_marginTop="8dp"
  }
}

-- 创建适配器
local adapter = LuaAdapter(activity, data, item)
listView.setAdapter(adapter)

-- 点击事件
listView.onItemClick = function(l, v, p, i)
  local item = data[p + 1]
  print("点击了:", item.title)
end
]]
      editor.pasteText(code)
      EditView.format()
    end
    
    -- 生成HTTP请求
    function generateHttpRequest()
      local code = [[
-- HTTP GET 请求
Http.get("https://api.example.com/data", nil, nil, nil, function(code, content, cookie, header)
  if code == 200 and content then
    -- 请求成功
    print("响应数据:", content)
    
    -- 解析JSON (如果需要)
    local data = require("cjson").decode(content)
    
  else
    -- 请求失败
    print("请求失败, 状态码:", code)
  end
end)
]]
      editor.pasteText(code)
      EditView.format()
    end
  end
}
```

## 发布与调试

### 1. 调试技巧

```lua
-- 使用 print 输出调试信息
print("调试信息:", variable)

-- 使用 Toast 显示消息
Toast.makeText(activity, "调试消息", Toast.LENGTH_SHORT).show()

-- 错误处理
local success, error = pcall(function()
  -- 可能出错的代码
end)

if not success then
  print("错误:", error)
  Toast.makeText(activity, "发生错误: " .. error, Toast.LENGTH_LONG).show()
end
```

### 2. 性能优化

- 避免在主线程进行耗时操作
- 使用 `activity.newTask` 进行异步处理
- 合理使用缓存机制
- 及时释放不需要的资源

### 3. 发布准备

1. **测试功能**: 确保所有功能正常工作
2. **检查兼容性**: 在不同版本的 LuaAppX 上测试
3. **优化性能**: 检查内存使用和响应速度
4. **完善文档**: 编写使用说明和更新日志

### 4. 目录打包

将插件文件夹整体复制到 LuaAppX 的插件目录即可安装使用。

## 最佳实践

1. **代码组织**: 将不同功能分离到不同的函数中
2. **错误处理**: 总是添加适当的错误处理
3. **用户体验**: 提供清晰的提示和反馈
4. **性能考虑**: 避免阻塞主线程
5. **兼容性**: 考虑不同版本和设备的兼容性

## 总结

通过本教程，您应该能够:

- 理解 LuaAppX 插件的基本结构
- 创建自定义的功能按钮和菜单
- 设计用户友好的界面
- 集成实用的工具功能
- 实现代码模板系统
- 处理网络请求和文件操作

插件开发是一个渐进的过程，建议从简单功能开始，逐步增加复杂性。参考现有的优秀插件(如快捷工具栏v3)可以获得更多灵感和最佳实践。

---

**注意**: 本文档基于 LuaAppX2 版本编写，不同版本可能存在API差异，请参考相应版本的官方文档。
