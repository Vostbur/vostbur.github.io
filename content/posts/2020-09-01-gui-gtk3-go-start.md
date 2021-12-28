---
categories:
- Programming
date: "2020-09-01"
description: Windows GUI-app on Golang with GTK 3.
slug: windows-gui-app-golang-gtk
tags:
- golang
- gtk3
- gui
- windows
title: Windows GUI-app on Golang with GTK 3
---

To develop a GUI app in Windows, I chose the [gotk3](https://github.com/gotk3/gotk3) package. This is a binding of the GTK lib, which provides a native interface for Windows.

I didn't deal with crosscompiling and installed all the necessary tools in the Windows 10 VM. The VM image is officially [distributed by Microsoft](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/).

I used the instructions from the [wiki](https://github.com/gotk3/gotk3/wiki/Installing-on-Windows) to install *gotk3*.

Also I used the [Chocolatey](https://chocolatey.org/install) to install the necessary tools. For install it:

1. Start PowerShell with Admin privilege
2. Download and run Chocolatey install script [https://chocolatey.org/install.ps1](https://chocolatey.org/install.ps1)

Then

```
PS C:\> choco install golang
PS C:\> choco install git
PS C:\> choco install msys2
PS C:\> choco install vscode
```

Close PowerShell then run again at user mode for confirm PATH settings. Exec:

```
PS C:\> mingw64
$ pacman -S mingw-w64-x86_64-gtk3 mingw-w64-x86_64-glade mingw-w64-x86_64-toolchain base-devel glib2-devel
$ echo 'export PATH=/c/Go/bin:$PATH' >> ~/.bashrc
$ echo 'export PATH=/c/Program\ Files/Git/bin:$PATH' >> ~/.bashrc
$ source ~/.bashrc
$ sed -i -e 's/-Wl,-luuid/-luuid/g' /mingw64/lib/pkgconfig/gdk-3.0.pc # This fixes a bug in pkgconfig
$ go get github.com/gotk3/gotk3/gtk
```

Glade is a GUI designer for GTK. It lets you design your GUI and export it in XML format. You can start glade.exe from `C:\tools\msys64\mingw64\bin\glade.exe`

Let's create a test app

```
$ mkdir $GOPATH/src/gtk-exmpl1
$ cd $GOPATH/src/gtk-exmpl1
```

File `main.go`

```
package main

import (
	"log"

	"github.com/gotk3/gotk3/gtk"
)

func main() {
	gtk.Init(nil)
	win, err := gtk.WindowNew(gtk.WINDOW_TOPLEVEL)
	if err != nil {
		log.Fatal("Unable to create window:", err)
	}
	win.SetTitle("Simple Example")
	win.Connect("destroy", func() {
		gtk.MainQuit()
	})
	l, err := gtk.LabelNew("Hello, gotk3!")
	if err != nil {
		log.Fatal("Unable to create label:", err)
	}
	win.Add(l)
	win.SetDefaultSize(800, 600)
	win.ShowAll()
	gtk.Main()
}
```
The last step is compilation

```
$ go build main.go
```

Enjoy the results!

![](/images/go-gtk-win-exmpl.PNG)
