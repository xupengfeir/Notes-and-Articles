Git使用：

1、直接拖动上传。

（1）打开远程仓库点击upload。

（2）点击或拖动文件/文件夹上传。

2、git上传

（1）新建仓库

git init		// git初始化

git add README.md	// 文件添加到暂存区

git commit -m "first commit"	//暂存区提交本地git仓库

git branch -M main	// 切换提交分支，自行定义分支名字

git remote add origin [https://github.com/](https://github.com/xupengfeir/Bayesian-Optimization-for-)xxxxx.git		//本地库添加到远程库（你的gihub仓库）

git push -u origin main		//推送到远程仓库



（2）在原本地仓库新增的文件

命令行步骤：

1）git init

​	2）git add [files] / .	(files代表选定的文件； . 代表当前目录所有文件)

​	3）git commit -m “文件上传注释”

​	4）git push origin main



（3）在其他文件内的文件

​	git init

​	git pull [https://github.com/](https://github.com/xupengfeir/Bayesian-Optimization-for-)xxxxx.git

​	git add [files] / .	(files代表选定的文件； . 代表当前目录所有文件)

​	git commit -m “文件上传注释”

​	git branch -M main

​	git remote add origin [https://github.com/](https://github.com/xupengfeir/Bayesian-Optimization-for-)xxxxx.git

​	git push -u origin main



3、README.md添加图片

（1）在github上的仓库建立一个存放图片的文件夹，文件夹名字随意。如：img-folder

（2）将需要在READNE.md中显示的图片，push到img-folder文件夹中。

（3）然后打开github官网，进入仓库的img-folder文件夹中，打开图片。

（4）复制图片地址链接。

（5）在README.md中填入并保存

![Image text](图片地址)

注：![Image text]这个标识不可缺少，不然就显示文字了。

Image text：指的是若图片不存在，要显示的文字说明。