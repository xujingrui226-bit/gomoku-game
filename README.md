<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <!-- 关键：添加viewport元标签，确保移动端正确缩放 -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>响应式设计示例</title>
    <style>
        /* 基础样式 - 适用于所有设备 */
        .container {
            width: 100%;
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
            box-sizing: border-box;
        }
        
        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 15px 0;
            border-bottom: 1px solid #eee;
        }
        
        .logo {
            font-size: 24px;
            font-weight: bold;
        }
        
        .nav {
            display: flex;
            gap: 30px;
        }
        
        .nav a {
            text-decoration: none;
            color: #333;
        }
        
        .content {
            display: flex;
            gap: 30px;
            margin-top: 30px;
        }
        
        .main-content {
            flex: 3;
        }
        
        .sidebar {
            flex: 1;
        }
        
        /* 响应式设计 - 手机适配 */
        /* 当屏幕宽度小于768px时（典型手机屏幕） */
        @media (max-width: 768px) {
            .container {
                padding: 10px;
            }
            
            .header {
                flex-direction: column;
                gap: 15px;
                padding: 10px 0;
            }
            
            .nav {
                width: 100%;
                justify-content: space-around;
                gap: 10px;
            }
            
            /* 在手机上改为垂直布局 */
            .content {
                flex-direction: column;
                gap: 20px;
            }
            
            /* 调整字体大小适应小屏幕 */
            .logo {
                font-size: 20px;
            }
            
            /* 确保图片自适应 */
            img {
                max-width: 100%;
                height: auto;
            }
        }
        
        /* 针对更小的手机屏幕（如375px以下） */
        @media (max-width: 375px) {
            .nav {
                flex-wrap: wrap;
            }
            
            .nav a {
                font-size: 14px;
                margin-bottom: 5px;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <div class="logo">网站标题</div>
            <div class="nav">
                <a href="#">首页</a>
                <a href="#">新闻</a>
                <a href="#">产品</a>
                <a href="#">关于我们</a>
            </div>
        </div>
        
        <div class="content">
            <div class="main-content">
                <h2>主要内容</h2>
                <p>这是网页的主要内容区域...</p>
                <img src="your-image.jpg" alt="示例图片">
            </div>
            
            <div class="sidebar">
                <h3>侧边栏</h3>
                <p>这是侧边栏内容...</p>
            </div>
        </div>
    </div>
</body>
</html>
