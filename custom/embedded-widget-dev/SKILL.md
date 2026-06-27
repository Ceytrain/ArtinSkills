---
name: embedded-widget-dev
description: 嵌入式Widget开发指南 - Device端(ARM Cortex-M)和PC端(Qt)独立Widget组件的完整开发流程、协议设计模式和调试方法。
---

# 嵌入式Widget开发指南

本文档描述如何在嵌入式Device端和PC端创建独立的Widget组件的通用方法。

## 核心流程概览

```
PCApp (Qt Designer)                    Device (ARM Cortex-M)
┌─────────────────────┐                ┌─────────────────────┐
│ 1. 创建PC端Widget   │  generate       │ 1. 创建Device端    │
│ 2. 配置属性         │ ──────────────> │    Widget实现       │
│ 3. buildImageData   │  protocol CMD  │ 2. 解析协议CMD      │
│    生成协议数据     │  (byte array)  │ 3. 调用显示函数     │
└─────────────────────┘                └─────────────────────┘
```

## 开发步骤

### Step 1: 统一Widget Type ID

**原则**: Device和PCApp使用相同的ID号

```c
// Device - 在widget类型枚举中添加
typedef enum {
    widgetTypeBackground = 0,
    widgetTypeImage = 1,
    widgetTypeNumber = 10,
    // ...
    widgetTypeMyWidget = 23,  // 新增
} widgetTypeEnum;
```

```cpp
// PCApp - 在global.h中定义
#define WIDGET_TYPE_MYWIDGET  23
```

### Step 2: Device端实现

#### 2.1 创建Widget头文件

```c
// ldMyWidget.h
#ifndef _LD_MYWIDGET_H_
#define _LD_MYWIDGET_H_

#include "ldCommon.h"

typedef struct {
    ldWidget widget;
    ldHorizontalAlign hAlign:2;
    // 自定义成员
    uint8_t displayType;
    uint8_t portIdx;
    float refreshValue;
} ldMyWidget;

ldMyWidget* ldMyWidgetInit(uint16_t nameId, uint16_t parentNameId, 
    uint16_t x, uint16_t y, int16_t width, int16_t height, ...);
void ldMyWidgetLoop(ldMyWidget *info, const arm_2d_tile_t *ptParent, bool bIsNewFrame);

#endif
```

#### 2.2 创建Widget实现文件

```c
// ldMyWidget.c
#include "ldMyWidget.h"

ldMyWidget* ldMyWidgetInit(uint16_t nameId, uint16_t parentNameId, 
    uint16_t x, uint16_t y, int16_t width, int16_t height, ...)
{
    ldMyWidget *pNewWidget = LD_MALLOC_WIDGET_INFO(ldMyWidget);
    if(pNewWidget == NULL) return NULL;
    
    pNewWidget->widgetType = widgetTypeMyWidget;
    pNewWidget->hAlign = hCenter;
    
    // parentNameId=0时链接到Widget链表
    if(parentNameId == 0) {
        ldWidgetLink((ldWidget*)pNewWidget);
    }
    
    return pNewWidget;
}

void ldMyWidgetLoop(ldMyWidget *info, const arm_2d_tile_t *ptParent, bool bIsNewFrame)
{
    // 显示逻辑
    switch(info->hAlign) {
        case hLeft:   break;
        case hCenter: break;
        case hRight:  break;
    }
}
```

#### 2.3 注册到显示循环

在 `ldCommon.c` 的 `ldCommonLoop` 函数中添加:
```c
case widgetTypeMyWidget:
    ldMyWidgetLoop((ldMyWidget*)pWidget, ptTile, bIsNewFrame);
    break;
```

#### 2.4 定义协议CMD

在 `global.h` 中定义协议索引，**对齐字段放末尾**:
```c
// 基础字段从11开始
#define CMD_MY_DISPLAY_TYPE   11
#define CMD_MY_PORT_ID         12
#define CMD_MY_DECIMAL         13
// ... 更多字段

// 对齐字段放末尾!
#define CMD_MY_ALIGN           45
```

#### 2.5 解析协议创建Widget

在 `runScene.c` 的 widget 创建 switch 中添加:
```c
case widgetTypeMyWidget:
{
    uint8_t hAlign = cmd[CMD_MY_ALIGN];  // 从末尾读取
    
    ldMyWidget *pMy = ldMyWidgetInit(newId, 0,
        CONNECT16(cmd[CMD_X_H], cmd[CMD_X_L]),
        CONNECT16(cmd[CMD_Y_H], cmd[CMD_Y_L]),
        ...);
    
    if(pMy != NULL) {
        pMy->hAlign = hAlign;
    }
    break;
}
```

### Step 3: PCApp端实现

#### 3.1 创建Widget类

继承WidgetBase，实现以下方法:
- showProperty() - 显示属性面板
- onPropertyChanged() - 属性变更回调
- xmlParse() / xmlBuild() - XML序列化
- getInfo() / setInfo() - 数据交互

#### 3.2 实现getInfo方法

**关键**: 返回顺序需与协议CMD对应
```cpp
QVariantList LlMyWidget::getInfo(QVariantList retList)
{
    retList.append((uint8_t)widgetTypeMyWidget);  // type
    retList.append((uint8_t)id + 1);               // id
    retList.append(x);
    retList.append(y);
    retList.append(w);
    retList.append(h);
    retList.append((uint8_t)displayType);
    retList.append((uint8_t)portIdx);
    // ... 其他参数
    retList.append(hAlign);  // 对齐放末尾
    return retList;
}
```

#### 3.3 实现buildImageData

在 `designer.cpp` 中添加协议生成逻辑:
```cpp
case widgetTypeMyWidget:
{
    tempByteArray[0] = (char)(0x80 + varListInfo.at(0).toInt());
    tempByteArray[1] = varListInfo.at(1).toInt();
    // ... 设置基础字段
    
    // 对齐字段放末尾索引45
    tempByteArray[45] = varListInfo.at(12).toInt();
    
    widgetCmdData.append(tempByteArray);
    break;
}
```

## 关键技术模式

### 模式1: 对齐字段放末尾

**原因**: 在协议中间插入字段会导致后续所有字段错位

**正确做法**:
```c
// ❌ 错误 - 在中间插入
#define CMD_MY_FIELD1   11
#define CMD_MY_ALIGN     12  // 错位!
#define CMD_MY_FIELD2    13

// ✅ 正确 - 放末尾
#define CMD_MY_FIELD1    11
#define CMD_MY_FIELD2    12
// ...
#define CMD_MY_ALIGN     45  // 末尾
```

### 模式2: 每个字符独立宽度

需要显示多位字符(如数字)时:

**PCApp端**:
```cpp
int16_t fontWArr[11] = {0};  // 0-9 + 小数点
QStringList charStrList = QStringList() << "0" << "1" << ... << ".";

for(int idx = 0; idx < charStrList.size(); idx++) {
    foreach (auto charInfo, imgMyWidgetCharInfoWriteList) {
        if(charInfo.charStr == charStrList.at(idx)) {
            fontWArr[idx] = charInfo.width;
        }
    }
}
// 输出到协议 (每个字符2字节)
for(int i = 0; i < 11; i++) {
    tempByteArray[23 + i*2] = GET16H(fontWArr[i]);
    tempByteArray[24 + i*2] = GET16L(fontWArr[i]);
}
```

**Device端**:
```c
fontW = CONNECT16(cmd[CMD_MY_FONT_W0_H], cmd[CMD_MY_FONT_W0_L]);
pMyWidgetSetImage(pMy, 0, '0', fontW, fontH, addrOffset);
```

### 模式3: 独立Font List

不同Widget使用独立的font list，避免冲突:
```cpp
QString fontName = fontSizeName + "MyWidget";  // 独立后缀
QList<imgCharInfoWrite_t> imgMyWidgetCharInfoWriteList;
```

### 模式4: Widget链表链接

当没有父Widget时，链接到主链表:
```c
if(parentNameId == 0) {
    ldWidgetLink((ldWidget*)pNewWidget);
}
```

## 调试方法

### 1. 协议数据检查

```cpp
// PCApp - 输出hex查看协议
qDebug() << widgetCmdData.toHex(' ').toUpper();
```

### 2. 数据完整性检查

```cpp
if(varListInfo.count() < 预期数量) {
    qDebug() << "MyWidget count error:" << varListInfo.count();
}
```

### 3. 字体信息调试

```cpp
qDebug() << "MyWidget fontH:" << fontH << "fontW arr:" 
         << fontWArr[0] << fontWArr[1] << fontWArr[2];
```

### 4. 枚举值检查

```c
// Device - 确认枚举值
switch(info->hAlign) {
    case hLeft:   qDebug("Left"); break;
    case hCenter: qDebug("Center"); break;
    case hRight:  qDebug("Right"); break;
}
```

## 常见问题与解决方案

### 问题1: 显示错位
**原因**: 协议索引不匹配，中间插入字段
**解决**: 对齐字段放末尾，检查CMD索引连续性

### 问题2: 对齐反向
**原因**: hAlign读取位置错误
**解决**: 
1. 确认从CMD_MY_ALIGN读取(非中间位置)
2. 检查ldHorizontalAlign枚举定义

### 问题3: 字符宽度不对
**原因**: 使用了错误的font list
**解决**: 
1. 确认使用独立的imgMyWidgetCharInfoWriteList
2. 检查font list后缀是否正确

### 问题4: 编译报错redeclaration
**原因**: 变量重复声明或作用域问题
**解决**: 检查大括号匹配，移除重复声明

### 问题5: 数据不显示
**检查清单**:
1. parentNameId=0时是否调用ldWidgetLink
2. CMD索引是否与PCApp对应
3. 内存分配是否成功
4. 字体图片地址是否正确

## 文件修改清单(通用)

| 层级 | 需要修改的文件 | 内容 |
|-----|--------------|------|
| Device | ldCommon.h | 添加widgetTypeMyWidget枚举 |
| Device | ldCommon.c | 添加widgetTypeMyWidget case |
| Device | ldMyWidget.h | 新建Widget头文件 |
| Device | ldMyWidget.c | 新建Widget实现文件 |
| Device | global.h | 添加CMD_MY_*协议定义 |
| Device | runScene.c | 添加widgetTypeMyWidget解析逻辑 |
| PCApp | designer/global.h | 添加WIDGET_TYPE_MYWIDGET |
| PCApp | widgets/llmywidget.h | 新建Widget类头文件 |
| PCApp | widgets/llmywidget.cpp | 新建Widget类实现 |
| PCApp | designer.cpp | 添加buildImageData case |

## 开发检查表

- [ ] 确定Widget Type ID (Device与PCApp一致)
- [ ] 创建ldMyWidget.h/c
- [ ] 注册到ldCommonLoop
- [ ] 定义协议CMD (对齐放末尾)
- [ ] 实现runScene解析逻辑
- [ ] 创建PC端llmywidget.h/c
- [ ] 实现getInfo方法
- [ ] 实现buildImageData
- [ ] 测试显示
- [ ] 测试对齐功能
- [ ] 测试各字符宽度
