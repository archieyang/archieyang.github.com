---
layout: post
title: "在第二台机器上部署Octopress"
date: 2013-11-25 22:57:45 +0800
comments: true
categories: Octopress
tags: [Octopress]
---
1. 首先clone source分枝到本地

		$ git clone -b source git@github.com:username/username.github.com.git octopress

2. 进入octopress文件夹


		$ cd octopress


3. 使用rake命令配置octopress，并输入repository的地址


		$ gem install bundler
		$ bundle install
		$ rake setup_github_pages
<!-- more -->

	

4. clone master分枝到_deploy

		$ rm -rf _deploy			#remove the generated _deploy
		$ git clone git@github.com:username/username.github.com.git _deploy 

5. 生成并发布

		$ rake generate
		$ git add .
		$ git commit -am "Some comment here." 
		$ git push origin source  # update the remote source branch 
		$ rake deploy             # update the remote master branch

6. 在另一台机器修改后，在本台机器修改前，pull source和master到octopress和_deploy

		$ cd octopress
		$ git pull origin source  # update the local source branch
		$ cd _deploy
		$ git pull origin master  # update the local master branch
