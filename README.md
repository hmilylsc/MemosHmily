# MemosHmily
简介：通过Memos的API新建的Hexo页面：在Memos上发布新内容，即可在Hexo博客页面同步显示。

🌸展示效果参考的朋友圈样式
🌸时间显示也是参考的朋友圈逻辑
🌸图片最多支持9宫格显示还是参考的朋友圈逻辑



## Hexo使用方法

1.在source文件夹📂新建memos文件夹

2.下载 index.md 到 memos文件夹内

3.hexo g && hexo s 

4.访问 http://localhost:4000/memos/ 查看效果



## 需要修改的内容

**目标代码在 177 - 181 行**

1.数据源地址 Memos 服务端地址。【替换成自己的 Memos 地址】
host: 'https://life.hmily.ren'

2.单次加载数量。
pageSize: 10

3.头像链接
avatarUrl: 'https://lsc.hmily.ren/image/260.png'

4.显示的昵称
nickname: 'Cookie'

5.开启图片灯箱效果（大图浏览）
enableFancybox: true
