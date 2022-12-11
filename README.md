# ESXi 安装镜像封装驱动程序
在 Windows 11 中给 VMware vSphere Hypervisor (ESXi) 安装镜像封装驱动程序，以下以 ESXi 6.7 为例。

## 下载
- [VMware vSphere Hypervisor (ESXi) 6.7](https://my.vmware.com/cn/group/vmware/patch)
  - 登录；
  - 选择 *ESXi Embedded and Installable* 和 *6.7.0*，搜索；
  - 下载最新版，目前为 **ESXi670-202210001.zip**。
- [VMware PowerCLI 6.5](https://code.vmware.com/web/tool/6.5%20R1/vmware-powercli)
- [ESXi-Customizer-PS-v2.6.0.ps1](https://www.v-front.de/p/esxi-customizer-ps.html)
- [net55-r8168 驱动程序](https://vibsdepot.v-front.de/wiki/index.php/Net55-r8168) net55-r8168-8.045a-napi-offline_bundle.zip
- [esxui-offline-bundle-6.x-12086396.zip](https://flings.vmware.com/esxi-embedded-host-client)

## 安装 VMware PowerCLI 6.5
1. 安装程序点右键，以管理员身份运行；
2. 以**管理员身份**运行 PowerCLI；
3. 首次运行时，需在 VMware PowerShell 中执行`Set-ExecutionPolicy Unrestricted`，输入 *Y* 确认。

## 添加单个驱动
以 net55-r8168 网卡驱动为例。
1. 将 VMware vSphere Hypervisor (ESXi) 6.7、ESXi-Customizer-PS-v2.6.0.ps1 与 net55-r8168 驱动程序置于同一目录；
2. 在 VMware PowerCLI 中该目录下执行

```PowerShell
.\ESXi-Customizer-PS-v2.6.0.ps1 -izip .\ESXi670-202210001.zip -dpt .\net55-r8168-8.045a-napi-offline_bundle.zip -load net55-r8168
```

3. 如提示 *Could not find a trusted signer*，则增加 **-nsc** 参数，即

```PowerShell
.\ESXi-Customizer-PS-v2.6.0.ps1 -izip .\ESXi670-202210001.zip -dpt .\net55-r8168-8.045a-napi-offline_bundle.zip -load net55-r8168 -nsc
```

4. 待提示 *All done*，则表明驱动程序已封装成功，上述目录下已生成 **ESXi-6.7.0-20221004001-standard-customized.iso** 文件。

5. 由于 ESXi670-202210001.zip 自带的 VMware Host Client v1.43.10 使用有问题，故要降级到低版本 v1.33.1。将 esxui-offline-bundle-6.x-12086396.zip 上传到 datastore1 目录，通过 SSH 登录 ESXi，执行以下命令

```sh
esxcli software vib install -d /vmfs/volumes/datastore1/esxui-offline-bundle-6.x-12086396.zip
```

## 添加多个驱动
1. 将下载到的各个驱动程序 zip 文件解压，分别得到一个 *vib* 文件；
2. 将 VMware vSphere Hypervisor (ESXi) 6.7、ESXi-Customizer-PS-v2.6.0.ps1 置于同一目录，将所有 *.vib* 文件置于该目录的一个子目录 vibs 中；
3. 在 VMware PowerCLI 中的 ESXi-Customizer-PS-v2.6.0.ps1 所在目录中执行

```PowerShell
.\ESXi-Customizer-PS-v2.6.0.ps1 -izip .\ESXi670-202210001.zip  -pkgDir .\vibs
```

4. 如提示 *Could not find a trusted signer*，则增加 **-nsc** 参数，即

```PowerShell
.\ESXi-Customizer-PS-v2.6.0.ps1 -izip .\ESXi670-202210001.zip  -pkgDir .\vibs -nsc
```

5. 待提示 *All done*，则表明驱动程序已封装成功，上述目录下已生成 **ESXi-6.7.0-20221004001-standard-customized.iso** 文件。

6. 由于 ESXi670-202210001.zip 自带的 VMware Host Client v1.43.10 使用有问题，故要降级到低版本 v1.33.1。将 esxui-offline-bundle-6.x-12086396.zip 上传到 datastore1 目录，通过 SSH 登录 ESXi，执行以下命令

```sh
esxcli software vib install -d /vmfs/volumes/datastore1/esxui-offline-bundle-6.x-12086396.zip
```
