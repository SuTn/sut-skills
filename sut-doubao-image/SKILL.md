---
name: sut-doubao-image
description: 使用豆包AI生成图片。当用户提到"豆包"、"豆包生成图片"、"用豆包画图"或指定豆包作为图片生成工具时触发此Skill。
---

# 豆包AI图片生成

使用豆包(Doubao) AI的图像生成功能创建高质量图片。

## 触发条件

当用户表达以下意图时使用此Skill：
- 明确提到"豆包"：如"用豆包生成图片"、"豆包画图"
- 指定豆包作为AI绘图工具
- 结合豆包和图片生成需求

## 调用步骤

### 1. 打开豆包网页

```javascript
await page.goto('https://www.doubao.com/chat/create-image');
await page.waitForTimeout(2000);
```

### 2. 设置图片比例（如果用户指定）

**重要：比例必须在输入提示词之前设置**

```javascript
// 如果用户指定了比例
if (用户指定了比例) {
  // 点击比例按钮
  const ratioButton = page.locator('button').filter({ hasText: '比例' });
  await ratioButton.click();
  await page.waitForTimeout(300);

  // 选择对应比例（1:1, 16:9, 9:16, 4:3, 3:4）
  await page.locator('generic').filter({ hasText: 用户指定的比例 }).click();
  await page.waitForTimeout(500);

  // 豆包会自动在输入框中添加比例提示词，如 "--比例 16:9"
}
```

### 3. 输入提示词（关键：保留比例提示词）

```javascript
const inputBox = page.locator('textbox').filter({ hasText: /描述你想要的图片/ });

if (用户指定了比例) {
  // 关键：获取当前输入框的值（包含比例提示词）
  const currentValue = await inputBox.inputValue();
  // 在比例提示词后面追加用户的提示词
  await inputBox.fill(currentValue + ' ' + 用户提示词);
} else {
  // 没有指定比例，直接输入提示词
  await inputBox.fill(用户提示词);
}

// 按回车生成
await inputBox.press('Enter');
```

### 4. 等待图片生成

```javascript
await page.waitForTimeout(5000);
```

### 5. 下载生成的图片

```javascript
// 获取所有生成的图片
const images = await page.locator('img[alt="image"]').all();

for (let i = 0; i < images.length; i++) {
  // 点击图片打开预览
  await images[i].click();
  await page.waitForTimeout(500);

  // 点击"下载原图"按钮
  const downloadButton = page.getByTestId('edit_image_download_button');
  await downloadButton.click();
  await page.waitForTimeout(1000);

  // 移动下载的文件到目标目录
  // 下载文件会被保存到 .playwright-mcp/生成图片.png

  // 如果不是最后一张，关闭预览继续下一张
  if (i < images.length - 1) {
    const closeButton = page.locator('.right-icon-b6Txpw > .semi-icon').first();
    await closeButton.click();
    await page.waitForTimeout(500);
  }
}
```

### 6. 保存图片到工作目录

```bash
# 使用通配符处理中文文件名编码问题
cp ".playwright-mcp/"*.png "工作目录/图片名-1.png"
rm ".playwright-mcp/"*.png
# 重复4次，每次改序号
```

## 关键注意事项

### 比例设置的正确流程

1. **先设置比例** → 豆包在输入框自动添加比例提示词（如 `--比例 16:9`）
2. **获取当前值** → `inputValue()` 获取包含比例的当前内容
3. **追加提示词** → `fill(currentValue + ' ' + prompt)` 保留比例提示词
4. **按回车生成** → 比例和提示词都生效

### 错误示例（比例不会生效）

```javascript
// 错误：直接fill会覆盖比例提示词
await inputBox.fill(用户提示词);
```

### 正确示例

```javascript
// 正确：保留比例提示词
const currentValue = await inputBox.inputValue();
await inputBox.fill(currentValue + ' ' + 用户提示词);
```

## 支持的比例

| 比例 | 说明 | 适用场景 |
|------|------|----------|
| 1:1 | 正方形 | 头像、社交媒体 |
| 16:9 | 横屏 | 横幅、风景 |
| 9:16 | 竖屏 | 手机壁纸、短视频 |
| 4:3 | 横屏 | 传统摄影 |
| 3:4 | 竖屏 | 传统人像 |

## 参数说明

| 参数 | 说明 | 默认值 |
|------|------|--------|
| 提示词 | 图片描述文字 | 必填 |
| 比例 | 图片宽高比 | 1:1 |
| 数量 | 生成图片数量 | 4张 |

## 完整示例代码

### 不带比例的图片生成

```javascript
await page.goto('https://www.doubao.com/chat/create-image');
await page.waitForTimeout(2000);

const inputBox = page.locator('textbox').filter({ hasText: /描述你想要的图片/ });
await inputBox.fill('一只可爱的猫咪');
await inputBox.press('Enter');
await page.waitForTimeout(5000);
```

### 带比例的图片生成（16:9）

```javascript
await page.goto('https://www.doubao.com/chat/create-image');
await page.waitForTimeout(2000);

// 设置比例
const ratioButton = page.locator('button').filter({ hasText: '比例' });
await ratioButton.click();
await page.waitForTimeout(300);
await page.locator('generic').filter({ hasText: '16:9' }).click();
await page.waitForTimeout(500);

// 输入提示词（保留比例提示词）
const inputBox = page.locator('textbox').filter({ hasText: /描述你想要的图片/ });
const currentValue = await inputBox.inputValue(); // 获取 "--比例 16:9"
await inputBox.fill(currentValue + ' 一只可爱的猫咪');
await inputBox.press('Enter');

await page.waitForTimeout(5000);
```

## 示例对话

**用户**: 用豆包生成一张黑神话悟空的图片

**执行**:
1. 打开豆包网页
2. 输入提示词"黑神话悟空"
3. 等待生成
4. 下载4张图片并保存到工作目录

**用户**: 豆包画一只猫，比例16:9

**执行**:
1. 打开豆包网页
2. 设置比例为16:9
3. 输入提示词（保留比例提示词）
4. 等待生成
5. 下载4张图片并保存

## 输出格式

生成的图片为PNG格式，分辨率通常为2048x2048（1:1比例）。
