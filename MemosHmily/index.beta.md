---
title: 碎片日志
date: 2099-09-09 21:00:00
comments: true
aside: false
---

<div id="memos-wrapper">
    <div id="memos-container">
        <div class="memos-loading">
            <div style="padding: 40px; color: #666; text-align: center;">日志加载中...</div>
        </div>
    </div>
    <div id="memos-load-more" style="display:none; text-align:center; margin-top:30px; margin-bottom: 20px;">
        <button class="load-btn">加载更多</button>
    </div>
</div>

<style>
/* --- 基础配置 --- */
/* --- 代码来自  https://www.hmily.ren --- */
.fancybox-container { z-index: 100000 !important; }



:root {
    --memo-avatar-size: 38px;
    --memo-gap: 8px;
    --memo-border: #f0f0f0;   
    --link-color: #576b95;    
    --text-main: #333;
    --text-sub: #999;
}

[data-theme="dark"] :root {
    --memo-border: #333;
    --link-color: #7d96cc;
    --text-main: #eee;
    --text-sub: #666;
}

/* --- 容器布局 --- */
#memos-wrapper {
    margin: 0 auto;
    font-family: -apple-system, BlinkMacSystemFont, sans-serif;
    box-sizing: border-box;
}

/* 移动端 (< 768px) */
@media (max-width: 767px) {
    #memos-wrapper {
        width: 100%;
        padding: 0 1px; 
        margin-top: 10px;
    }
}

/* PC 端 (>= 768px) */
@media (min-width: 768px) {
    #memos-wrapper {
        max-width: 800px; 
        width: 100%; 
        padding: 0 20px;
        margin-top: 20px;
    }
}

/* --- 条目样式 --- */
.memo-item {
    display: flex;
    gap: var(--memo-gap);
    padding: 16px 0;
    border-bottom: 1px solid var(--memo-border);
    animation: fadeIn 0.5s forwards;
    align-items: flex-start;
}
@keyframes fadeIn { to { opacity: 1; } }

/* 头像 */
.memo-avatar {
    flex-shrink: 0;
    width: var(--memo-avatar-size);
    height: var(--memo-avatar-size);
    margin: 0; 
}
.memo-avatar img {
    width: 100%;
    height: 100%;
    border-radius: 4px;
    object-fit: cover;
    background: #f0f0f0;
}

/* 内容区 */
.memo-content-wrapper {
    flex: 1;
    min-width: 0;
}

.memo-name {
    color: var(--link-color);
    font-weight: 600;
    font-size: 15px;
    margin-bottom: 2px;
    line-height: 1.2;
}

.memo-text {
    font-size: 15px;
    line-height: 1.5;
    color: var(--text-main);
    margin-bottom: 6px;
    white-space: pre-wrap;
    word-wrap: break-word;
    text-align: left;
	/* --- 新增：提示浏览器此元素的高度和内容可能会变，分配独立渲染层优化重绘【will-change: height;】 --- */
    
}

/* --- 新增：折叠展开功能相关 CSS --- */
.memo-text.collapsed {
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 5; /* 超出5行自动折叠并显示省略号 */
    overflow: hidden;
}
.memo-toggle-btn {
    color: var(--link-color);
    font-size: 15px;
    cursor: pointer;
    margin-bottom: 6px;
    margin-top: -2px;
    display: inline-block;
    user-select: none;
}
/* -------------------------------- */

.memo-text a { color: var(--link-color); text-decoration: none; }
.memo-text img { max-width: 100%; border-radius: 4px; display: block; margin: 5px 0; }

.memo-meta {
    font-size: 12px;
    color: var(--text-sub);
    margin-top: 6px;
}

/* --- 九宫格 --- */
.memo-gallery {
    display: grid;
    gap: 4px;
    grid-template-columns: repeat(3, 1fr);
    margin-top: 6px;
    max-width: 100%; 
}

@media (min-width: 768px) { .memo-gallery { max-width: 400px; } }

.img-box {
    position: relative;
    width: 100%;
    padding-bottom: 100%;
    background: #f0f0f0;
    cursor: zoom-in;
}
.img-box img {
    position: absolute; top: 0; left: 0; width: 100%; height: 100%;
    object-fit: cover; display: block;
}

/* 单图 */
.memo-gallery[data-count="1"] { display: block; }
.memo-gallery[data-count="1"] .img-box {
    width: auto; height: auto; padding-bottom: 0; background: transparent;
}
.memo-gallery[data-count="1"] img {
    position: relative;
    max-height: 350px;
    max-width: 75%; 
    width: auto; height: auto;
    border-radius: 4px;
    border: 1px solid rgba(0,0,0,0.05);
}

/* 2张/4张 */
.memo-gallery[data-count="2"], .memo-gallery[data-count="4"] {
    grid-template-columns: repeat(2, 1fr);
    width: 66%;
}

.load-btn {
    background: #fff; border: 1px solid #ddd; padding: 6px 30px; border-radius: 4px;
    color: #666; cursor: pointer; display: block; margin: 0 auto; font-size: 12px;
}
</style>

<script>
document.addEventListener('DOMContentLoaded', () => {
    const CONFIG = {
        host: 'https://life.hmily.ren',
        pageSize: 10,
        avatarUrl: 'https://lsc.hmily.ren/image/260.png',
        nickname: 'Cookie',
        enableFancybox: true
    };

    const container = document.getElementById('memos-container');
    const loadMoreBtn = document.getElementById('memos-load-more');
    const loadBtnEl = loadMoreBtn.querySelector('.load-btn');
    let nextPageToken = '';
    let isLoading = false;
	
	let lastScrollY = 0;

    function initFancybox() {
        if (!window.Fancybox) return;

        Fancybox.bind('[data-fancybox]', {
            animated: true,
            autoFocus: true,
            trapFocus: true,
            placeFocusBack: true,
            dragToClose: true,
            hideScrollbar: false,
            on: {
                init: () => {
                    lastScrollY = window.scrollY || window.pageYOffset;
                },
                destroy: () => {
                    requestAnimationFrame(() => {
                        window.scrollTo(0, lastScrollY);
                    });
                }
            }
        });
    }

    initFancybox();
    
    // 时间逻辑
    const formatTime = (isoString) => {
        const date = new Date(isoString).getTime();
        const currentDate = new Date().getTime();
        const spaceTime = Math.abs(currentDate - date) / 1000;
    
        if (spaceTime < 60) return '刚刚';
        if (spaceTime < 3600) return Math.floor(spaceTime / 60) + '分钟前';
        if (spaceTime < 86400) return Math.floor(spaceTime / 60 / 60) + '小时前';
        if (spaceTime < 172800) return '昨天';
        return Math.floor(spaceTime / 60 / 60 / 24) + '天前';
    };
    
    const parseContent = (t) => {
        if (!t) return '';
        t = t.replace(/\[([^\]]+)\]\(([^)]+)\)/g, '<a href="$2" target="_blank">$1</a>');
        t = t.replace(/(https?:\/\/[^\s]+)/g, (u) => u.includes('">')?u:`<a href="${u}" target="_blank">网页链接</a>`);
        return t.replace(/#(\S+)/g, '<span style="color:var(--link-color)">#$1</span>')
                .replace(/!\[([^\]]*)\]\(([^)]+)\)/g, '<img src="$2" loading="lazy">');
    };
    
    const loadMemos = async (pageToken = '') => {
        if (isLoading) return;
        isLoading = true;
    
        try {
            let url = `${CONFIG.host}/api/v1/memos?pageSize=${CONFIG.pageSize}`;
            if (pageToken) url += `&pageToken=${pageToken}`;
    
            const res = await fetch(url);
            if (!res.ok) throw new Error('网络请求失败');
            const data = await res.json();
            
            if (!pageToken) container.innerHTML = '';
            
            const items = data.memos || [];
            if (items.length === 0 && !pageToken) {
                container.innerHTML = '<div style="text-align:center;padding:40px">暂无动态</div>';
                return;
            }
    
            /* --- 性能优化：1. 先把所有 HTML 拼接成一个长字符串 --- */
            let htmlString = '';

            items.forEach((item) => {
                const attachments = item.attachments || [];
                const images = attachments.filter(res => res.type && res.type.startsWith('image/'));
    
                let galleryHtml = '';
                
                if (images.length > 0) {
                    let imgsHtml = '';
                    images.forEach(img => {
                        let src = img.externalLink;
                        if (!src) {
                            const filename = encodeURIComponent(img.filename || 'image.jpg');
                            src = `${CONFIG.host}/file/${img.name}/${filename}`;
                        }
                        
                        const thumbSrc = src + '?thumbnail=true';
                        const originalSrc = src;
                        const fancy = CONFIG.enableFancybox ? `data-fancybox="g-${item.name}"` : '';
                        
                        imgsHtml += `
                            <div class="img-box">
                                <a href="${originalSrc}" ${fancy} class="fancybox">
                                    <img src="${thumbSrc}" loading="lazy" referrerpolicy="no-referrer">
                                </a>
                            </div>
                        `;
                    });
                    galleryHtml = `<div class="memo-gallery" data-count="${images.length}">${imgsHtml}</div>`;
                }
    
                if (!item.content && images.length === 0) return;

                const rawContent = item.content || '';
    
                // 注意这里：全部默认带上 collapsed 类
                htmlString += `
                    <div class="memo-item">
                        <div class="memo-avatar"><img src="${CONFIG.avatarUrl}"></div>
                        <div class="memo-content-wrapper">
                            <div class="memo-name">${CONFIG.nickname}</div>
                            <div class="memo-text collapsed">${parseContent(rawContent)}</div>
                            ${galleryHtml}
                            <div class="memo-meta">${formatTime(item.createTime)}</div>
                        </div>
                    </div>`;
            });

            /* --- 性能优化：2. 一次性将拼接好的内容插入到页面中（只发生1次写入） --- */
            container.insertAdjacentHTML('beforeend', htmlString);

/* --- 性能优化：3. 统一查找并测量刚刚插入的元素（彻底消除布局抖动） --- */
const allCollapsedTexts = container.querySelectorAll('.memo-text.collapsed');

// 阶段一：纯读取阶段 (Gathering phase)
// 此时浏览器只需计算一次布局
const measureResults = [];
allCollapsedTexts.forEach(textEl => {
    // 如果已经处理过，跳过
    if (textEl.nextElementSibling && textEl.nextElementSibling.classList.contains('memo-toggle-btn')) {
        return;
    }
    
    // 集中【读】：此时不会有任何 DOM 修改打断布局缓存
    const isOverflowing = textEl.scrollHeight > textEl.clientHeight;
    measureResults.push({ el: textEl, isOverflowing });
});

// 阶段二：纯写入阶段 (Mutation phase)
// 集中修改 DOM，浏览器会将这些修改合并成一次绘制
measureResults.forEach(({ el, isOverflowing }) => {
    if (isOverflowing) {
        const btn = document.createElement('div');
        btn.className = 'memo-toggle-btn';
        btn.innerText = '全文';
        el.after(btn);
    } else {
        el.classList.remove('collapsed');
    }
});
/* ------------------------------------------------------------- */
    
            nextPageToken = data.nextPageToken;
            if (nextPageToken) {
                loadMoreBtn.style.display = 'block';
                loadBtnEl.innerText = '加载更多';
            } else {
                loadMoreBtn.style.display = 'none';
            }
    
        } catch (e) {
            console.error(e);
            if (!pageToken) container.innerHTML = '<div style="text-align:center;padding:20px">加载失败</div>';
            loadBtnEl.innerText = '重试';
        } finally {
            isLoading = false;
        }
    };
    
    loadBtnEl.addEventListener('click', () => {
        loadBtnEl.innerText = '加载中...';
        loadMemos(nextPageToken);
    });

    // 事件代理保持不变
    container.addEventListener('click', (e) => {
        if (e.target.classList.contains('memo-toggle-btn')) {
            const btn = e.target;
            const textEl = btn.previousElementSibling; 
            if (textEl.classList.contains('collapsed')) {
                textEl.classList.remove('collapsed');
                btn.innerText = '收起';
            } else {
                textEl.classList.add('collapsed');
                btn.innerText = '全文';
            }
        }
    });
    
    const wrapper = document.getElementById('memos-wrapper');
    if (wrapper) {
        const footer = document.createElement('div');
        footer.style.textAlign = 'center';
        footer.style.marginBottom = '30px';
        footer.style.fontSize = '12px';
        footer.style.color = 'var(--text-sub)'; 
        footer.innerHTML = '本页面由 <a href="https://github.com/hmilylsc/MemosHmily" target="_blank" style="color: var(--link-color); text-decoration: none;">MemosHmily</a> 提供';
        wrapper.appendChild(footer);
    }
    
    loadMemos();
});
</script>
