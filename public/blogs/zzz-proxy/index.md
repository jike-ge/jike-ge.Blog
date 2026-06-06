打开终端，输入以下命令关闭图形界面：

```shell
sudo systemctl disable lightdm.service
```

或者使用以下命令在启动时禁用图形界面：

```shell
sudo nano /etc/default/grub
```

将 GRUB_CMDLINE_LINUX_DEFAULT="quiet splash" 修改为 GRUB_CMDLINE_LINUX_DEFAULT="text"，然后更新grub配置：

```shell
sudo update-grub
```

如果上方无效的话，可以使用systemd设置默认运行级别：
查看当前的默认运行级别：

```shell
systemctl get-default
```

将默认运行级别设置为多用户模式（关闭图形界面）：

```shell
sudo systemctl set-default multi-user.target
```

需要重新启用图形界面时，可以使用以下命令：

```shell
sudo systemctl set-default graphical.target
```

操作完成后重启一下查看效果