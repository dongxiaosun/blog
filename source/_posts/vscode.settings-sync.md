title: 如何将 VS Code 的设置保存在 github 上（Setting Sync 使用）
date: 2019-04-26
abbrlink: vscode/settings-sync
categories:
  - 教程
tags:
  - VS Code

---

Setting Sync 插件是将本地编辑器配置保存到 GitHub 上(本地配置修改会实时上传)，以后只需要从 GitHub 上获取，就可以一次性安装插件配置信息。

<!-- more -->

## 安装

在活动面板“扩展”中搜索 Setting Sync 插件，并且安装。


+ Shift + Alt + U 快捷键备份(上传)
+ Shift + Alt + D 快捷键恢复(下载)

![](/assets/image/settingsync-2.jpg)

## 在 Github 上生成令牌（token）
按下 ```Shift + Alt + U``` 会打开浏览器进入 Github 界面

登陆 Github > settings > Developer settings > personal access tokens  > generate new token
输入名称，勾选Gist，提交，保存 **token（保存好）**

![](/assets/image/settingsync-1.jpg)

## 将生成的 token 输入 VS Code 中

![](/assets/image/settingsync-3.png)

## 保存生成的 Gist ID
**找到 Gist ID 并且保存起来**
![](/assets/image/settingsync-4.jpg)
![](/assets/image/settingsync-5.jpg)

## 恢复插件

按下 ```Shift + Alt + D``` 将 token 和 Gist ID 输入即可