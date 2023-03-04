- ## Arch Linux 安装 Docker 的步骤
	- 以下是在 Arch Linux 上安装 Docker 的步骤：
	  
	  1. 确认系统已更新
	  
	  在安装 Docker 之前，您需要确保系统已更新。为此，可以使用以下命令：
	  
	  ```
	  sudo pacman -Syu
	  ```
	  
	  2. 安装 Docker 
	  
	  执行以下命令安装 Docker：
	  
	  ```
	  sudo pacman -S docker
	  ```
	  
	  3. 启动 Docker 服务
	  
	  使用以下命令启动 Docker 服务：
	  
	  ```
	  sudo systemctl start docker
	  ```
	  
	  4. 将 Docker 添加到系统启动项
	  
	  使用以下命令将 Docker 添加到系统启动项：
	  
	  ```
	  sudo systemctl enable docker
	  ```
	  
	  5. 确认 Docker 是否已安装成功
	  
	  使用以下命令确认 Docker 是否已安装成功：
	  
	  ```
	  sudo docker run hello-world
	  ``` 
	  
	  如果输出信息中显示 "Hello from Docker!"，则说明 Docker 已成功安装。
	  
	  到此，您已经成功在 Arch Linux 上安装了 Docker！
-