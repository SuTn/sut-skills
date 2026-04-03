# 豆包图片生成实现指南

## 重要说明：比例设置问题

**问题**：当先选择比例后，豆包会在输入框中自动添加比例提示词（如 `--比例 16:9`）。如果直接使用 `fill()` 填入用户提示词，会覆盖掉这个比例提示词，导致比例设置失效。

**解决方案**：使用 `inputValue()` 获取当前输入框内容，然后在后面追加用户提示词。

## 完整实现流程

### Step 1: 打开浏览器并导航

```javascript
await page.goto('https://www.doubao.com/chat/create-image');
await page.waitForTimeout(2000);
```

### Step 2: 设置比例（如果用户指定）

```javascript
// 如果用户指定了比例，必须先设置比例
if (用户指定了比例) {
  const ratioButton = page.locator('button').filter({ hasText: '比例' });
  await ratioButton.click();
  await page.waitForTimeout(300);

  // 选择对应比例
  await page.locator('generic').filter({ hasText: 用户指定的比例 }).click();
  await page.waitForTimeout(500);

  // 豆包会自动在输入框添加比例提示词，如 "--比例 16:9"
}
```

### Step 3: 输入提示词（关键步骤）

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

await inputBox.press('Enter');
```

### Step 4: 等待图片生成

```javascript
await page.waitForTimeout(5000);
```

### Step 5: 下载所有图片

```javascript
const images = await page.locator('img[alt="image"]').all();

for (let i = 0; i < images.length; i++) {
  await images[i].click();
  await page.waitForTimeout(500);

  await page.getByTestId('edit_image_download_button').click();
  await page.waitForTimeout(1000);

  // 移动文件到工作目录
  // 使用bash命令处理中文文件名

  if (i < images.length - 1) {
    await page.locator('.right-icon-b6Txpw > .semi-icon').first().click();
    await page.waitForTimeout(500);
  }
}
```

### Step 6: 移动文件到工作目录

```bash
# 使用通配符处理中文文件名编码问题
cp ".playwright-mcp/"*.png "工作目录/图片名-1.png"
rm ".playwright-mcp/"*.png
```

## 完整代码示例

### 不带比例的图片生成

```javascript
await page.goto('https://www.doubao.com/chat/create-image');
await page.waitForTimeout(2000);

const inputBox = page.locator('textbox').filter({ hasText: /描述你想要的图片/ });
await inputBox.fill('一只可爱的猫咪');
await inputBox.press('Enter');

await page.waitForTimeout(5000);
// 下载图片...
```

### 带比例的图片生成（正确做法）

```javascript
await page.goto('https://www.doubao.com/chat/create-image');
await page.waitForTimeout(2000);

// 1. 先设置比例
const ratioButton = page.locator('button').filter({ hasText: '比例' });
await ratioButton.click();
await page.waitForTimeout(300);
await page.locator('generic').filter({ hasText: '16:9' }).click();
await page.waitForTimeout(500);

// 2. 输入提示词（保留比例提示词）
const inputBox = page.locator('textbox').filter({ hasText: /描述你想要的图片/ });
const currentValue = await inputBox.inputValue(); // 获取 "--比例 16:9"
await inputBox.fill(currentValue + ' 一只可爱的猫咪');
await inputBox.press('Enter');

await page.waitForTimeout(5000);
// 下载图片...
```

### 带比例的图片生成（错误做法❌）

```javascript
// 错误：这样会覆盖掉比例提示词
const inputBox = page.locator('textbox').filter({ hasText: /描述你想要的图片/ });
await inputBox.fill('一只可爱的猫咪'); // 直接fill会覆盖 "--比例 16:9"
await inputBox.press('Enter');
// 结果：比例设置失效，生成的是1:1的图片
```

## 常见问题处理

### 问题1: 比例没有生效

**原因**：直接使用 `fill()` 覆盖了比例提示词

**解决**：
```javascript
const currentValue = await inputBox.inputValue();
await inputBox.fill(currentValue + ' ' + 用户提示词);
```

### 问题2: 文件名编码问题

```bash
# 使用通配符代替中文文件名
mv ".playwright-mcp/"*.png "target.png"
```

### 问题3: 图片未加载完成

```javascript
// 增加等待时间
await page.waitForTimeout(8000);
```

### 问题4: 下载按钮未找到

```javascript
// 先获取快照确认元素存在
await page.snapshot();
// 使用多种选择器尝试
const downloadButton = page.getByTestId('edit_image_download_button')
  .or(page.locator('button').filter({ hasText: '下载原图' }));
```

## 支持的比例

| 比例 | 说明 | 比例提示词 |
|------|------|-----------|
| 1:1 | 正方形（默认） | --比例 1:1 |
| 16:9 | 横屏 | --比例 16:9 |
| 9:16 | 竖屏 | --比例 9:16 |
| 4:3 | 横屏 | --比例 4:3 |
| 3:4 | 竖屏 | --比例 3:4 |

## 实际调用示例

### 示例1：生成正方形图片（默认）

**用户**: "用豆包生成一只猫"

**执行**:
```javascript
await page.goto('https://www.doubao.com/chat/create-image');
await page.waitForTimeout(2000);
const inputBox = page.locator('textbox').filter({ hasText: /描述你想要的图片/ });
await inputBox.fill('一只猫');
await inputBox.press('Enter');
await page.waitForTimeout(5000);
```

### 示例2：生成16:9横屏图片

**用户**: "用豆包生成一只猫，比例16:9"

**执行**:
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
const currentValue = await inputBox.inputValue();
await inputBox.fill(currentValue + ' 一只猫');
await inputBox.press('Enter');

await page.waitForTimeout(5000);
```

## 测试检查清单

- [ ] 不指定比例时生成1:1图片
- [ ] 指定16:9时生成横屏图片
- [ ] 指定9:16时生成竖屏图片
- [ ] 4张图片都能成功下载
- [ ] 文件正确保存到工作目录
- [ ] 中文文件名编码正确处理
