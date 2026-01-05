[快速搭建个人博客 —— 保姆级教程 | 攻城狮杰森](https://pdpeng.github.io/2022/01/19/setup-personal-blog/)
**Hexo 的文件结构**
![[Hexo file structure.png]]
- public 最终所见网页的所有内容
- node_modules 插件以及 hexo 所需 node.js 模块
- \_config.yml 站点配置文件，设定一些公开信息等
- package.json 应用程序信息，配置 hexo 运行所需 js 包
- scaffolds 模板文件夹，新建文章，会默认包含对应模板内容
- themes 存放主题文件，hexo 根据主题生成静态网页（速度贼快）
- source 用于存放用户资源（除  *posts* 文件夹，其余命名方式为 “\_ + 文件名”的文件被忽略）
# Hexo 添加 Post 链接

`{% post_link <file name(without .md)> <shown name> %}`
