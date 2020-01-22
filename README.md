# halocsidez.github.io
## 使用 Travis CI 自动构建 GitHub Pages
  - https://travis-ci.com/
  
  
Travis完成了`将gh-source的分支内容进行自动构建，并提交至master分支`的任务。我的配置文件见/.travis.yml，内容如下：
  
``` yaml
language: node_js
node_js: stable

addons: # Travis CI建议加的，自动更新api
  apt:
    update: true

cache:
  directories: 
  - node_modules # 缓存 node_modules
  
before_install:
- npm install hexo-cli -g

install:
- npm install # 初次安装，在CI环境中，执行安装npm依赖

# before_script: 

script:
- hexo clean
- hexo g # 执行 hexo generate，把文章编译到public中
- hexo g # 避免主页空白

after_success: # 执行script成功后，进入到public，把里面的代码提交到博客的gh-pages分支
- cd ./public
- git init
- git config user.name "halocsidez"
- git config user.email "csidez@163.com"
- git add .
- git commit -m "Travis CI Auto Builder"
- git push --force --quiet "https://${Travis_Token}@${GH_REF}" master:master

branches:
  only:
  - gh-source # CI 只针对分支 dev

env:
  global: # 全局变量，上面的提交到github的命令有用到
  - GH_REF: github.com/HaloCsidez/halocsidez.github.io.git
  - secure: 
# secure是自动生成的，执行`travis encrypt 'GH_TOKEN=${your_github_personal_access_token}' --add`
```


## 主题 material
  - https://github.com/fluid-dev/hexo-theme-fluid
## 搜索功能 
  - https://github.com/wzpan/hexo-generator-search
