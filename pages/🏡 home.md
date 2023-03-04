public:: true
alias:: Home

- Welcome To My Digital Garden
	- Those notes are mostly written for myself but I'm sharing them publicly.
	- I use Logseq to review what I #read and what I #see.
- Contacts
	- [Blog](https://aetherhjf.com)
- Arch Linux 如何安装 Docker?
	- 你可以按照以下步骤在 Arch Linux 上安装 Docker：
	  
	  1. 打开终端并更新系统：
	  
	  ```
	  sudo pacman -Syu
	  ```
	  
	  2. 安装 Docker：
	  
	  ```
	  sudo pacman -S docker
	  ```
	  
	  3. 启动 Docker 服务：
	  
	  ```
	  sudo systemctl start docker.service
	  ```
	  
	  4. 设置 Docker 开机自启：
	  
	  ```
	  sudo systemctl enable docker.service
	  ```
	  
	  5. 验证 Docker 是否安装成功，运行以下命令查看 Docker 版本号：
	  
	  ```
	  docker --version
	  ```
	  
	  安装完成后，你可以开始使用 Docker 来管理和运行容器。
- ---
- You can find all the templates that I use here: [[🖨 templates]]
-