---
name: dmxapi-image-generation
description: "使用 MiniMax (image-01) 和其他模型生成图片。支持文生图、图片编辑。当用户需要生成图片、AI绘图时使用此技能。使用方法：用户需要配置 DMXAPI_API_KEY 环境变量。"
---

# MiniMax Image Generation Skill

## 功能

- **文生图** - 根据文本描述生成图片
- 支持多种尺寸和比例

## 使用前提

用户需要：
1. MiniMax API Key（sk-cp-xxx 格式）
2. 确保套餐包含图片生成功能

## API 调用方法

```javascript
const https = require('https');
const fs = require('fs');

const apiKey = 'YOUR_DMXAPI_API_KEY'; // 环境变量 DMXAPI_API_KEY

const body = JSON.stringify({
  model: 'image-01',
  prompt: '描述内容',
  aspect_ratio: '16:9', // 支持 1:1, 16:9, 9:16 等
  response_format: 'base64'
});

const req = https.request({
  hostname: 'api.minimaxi.com',
  port: 443,
  path: '/v1/image_generation',
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer ' + apiKey
  }
}, res => {
  const chunks = [];
  res.on('data', d => chunks.push(d));
  res.on('end', () => {
    const parsed = JSON.parse(Buffer.concat(chunks).toString());
    // 返回格式是 image_urls（不是 base64）
    const imageUrl = parsed.data[0].image_urls[0];
    // 下载图片
    https.get(imageUrl, imgRes => {
      const imgChunks = [];
      imgRes.on('data', d => imgChunks.push(d));
      imgRes.on('end', () => {
        const imgBuf = Buffer.concat(imgChunks);
        fs.writeFileSync('output.jpg', imgBuf);
        console.log('SAVED: ' + imgBuf.length);
      });
    });
  });
});
req.write(body);
req.end();
```

**重要：** API 返回的是 `image_urls`（图片URL），不是 `base64`。需要先获取URL，再下载图片。

## 示例

生成小龙虾头像：
- model: image-01
- prompt: cute cartoon lobster mascot, kawaii style, orange color, big eyes
- aspect_ratio: 1:1

生成五星级酒店背景墙：
- model: image-01
- prompt: Luxury vintage mid-century modern hotel bedroom accent wall, 3.5 meters wide, 2.2 meters tall, dark emerald green and warm brass gold tones, floor-to-ceiling textured wall panel with art-deco patterns
- aspect_ratio: 16:9