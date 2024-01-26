# Install OptiSign on Ubuntu 22.04

In this guide we are going to see how to install all depencies of OptiSign and then excecute the application

## Install Fuse2
To use OptiSign we need to have fuse2 (not fuse3) on the system. The best way is via _libfuse2_ that can
coexist with fuse3.

First we need to enable universe _repository_ with the following command:
```shell
sudo add-apt-repository universe
```

After that we can install libfuse2 with:
```shell
sudo apt install libfuse2
```

## Install OptiSign
OptiSign is a AppImage package so we need to download the bundle accordingly to our architecture 
from [OptiSign Website](https://www.optisigns.com/download).

Then we need to give the AppImage execution permission via our current File Manager or via terminal with:
```shell
sudo chmod +x OptiSign-XYZ.Appimage
```

Now you can execute the application by double tapping on the package or via terminal and it will start up.