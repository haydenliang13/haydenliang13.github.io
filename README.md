
# Hayden's Blog

A simple and clean blog for words and photos.

### 安装Jekyll环境（Archlinux）

	# pacman -S ruby

在`~/.bashrc`添加以下代码：

	PATH="$(ruby -e 'print Gem.user_dir')/bin:$PATH"

重启session确保`$PATH`已被更新，安装Jekyll

	$ gem install jekyll
