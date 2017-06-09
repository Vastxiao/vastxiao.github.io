# blog init by hexo.

## hexo install

https://hexo.io/zh-cn/

依赖软件:
- Node.js
- Git

```sh
# 安装hexo:
npm install -g hexo-cli

# 备注:使用hexo命令时有一些node_modules报错,可以重新安装下这个hexo.
# 或需要移除报错modules目录
cd /TO/HEXOINITDIR
npm install hexo --save

# 建站:
hexo init <folder>
cd <folder>
npm install

# 使用git部署站点需要安装包:
cd <folder>  #必须进入 hexo建站后的目录(bloginit)
npm install hexo-deployer-git --save

# 安装hexo插件，支持markdown流程图
cd <folder>  #必须进入 hexo建站后的目录(bloginit)
npm install --save hexo-filter-flowchart  #支持markdown flow 代码
npm install --save hexo-filter-sequence   #支持markdown seq 代码

# 配置/部署博客:
hexo new       # Create a new post.
hexo clean     # Removed generated files and cache.
hexo generate  # Generate static files.
hexo server    # Start the server.
hexo deploy    # Deploy your website.
```

