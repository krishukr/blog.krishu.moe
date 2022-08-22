---
title: 更有效率地规划 Ingress 多重 Field
date: 2022-08-21 15:58:30
tags:
  - Ingress
categories: 攻略
---

Ingress 中赚取 AP 的最快方式是通过 Link 建立 Field。而[多重 Field](https://ingress.fandom.com/zh/wiki/Control_Field) 更是效率加倍。

于是如何有效率地拉多重 Field 便是一门学问。

但是我脑子不太好使，所以用电脑帮我算。

<!-- more -->

# 安装 maxfield

前天群友告诉我有个项目叫 maxfield，似乎本来是个 web 端应用，但现在打不开了。所幸后端是[开源](https://github.com/tvwenger/maxfield)的，我们可以自己跑。

*upd: 现在[网页](https://www.ingress-maxfield.com/)好了，其数据格式是完全一样的，你可以继续看到准备数据集的部分。*

首先 clone maxfield 的仓库。

```
git clone https://github.com/tvwenger/maxfield
cd maxfield
```

maxfield 生成 GIF 图的效率低下，且实际使用中似乎很少会去看 GIF，所以我们简单修改一下源码。

在 `maxfield/results.py` 中拉到最底下，将如下部分**注释**掉。

```python
#
# Generate GIF
#
fname = os.path.join(self.outdir, 'plan_movie.gif')
with imageio.get_writer(fname, mode='I', duration=0.5) as writer:
    for frame in frames:
        image = imageio.imread(frame)
        writer.append_data(image)
optimize(fname)
if self.verbose:
    print("GIF saved to {0}".format(fname))
    print()
```

无论你是什么操作系统，都*建议*建立虚拟环境。

具体操作方式参考[官方文档](https://docs.python.org/zh-cn/3.8/library/venv.html)，以下以 Ubuntu(bash) 为例。

```
sudo apt install python3.8-venv -y
python3 -m venv venv
source venv/bin/activate
```

参考[项目文档](https://github.com/tvwenger/maxfield/blob/master/README.md#installation)安装 maxfield。

```
python3 setup.py install
```

我的 Windows 10(Powershell) 使用如下命令：

```
.\venv\Scripts\Activate.ps1
python setup.py install
```

# 准备数据集

maxfield 需要规划区域内的所有 Portal 的信息。一个方便的做法是直接从 [Ingress Intel Map](https://intel.ingress.com/) 导出。

首先安装 IITC，见[文档](https://iitc.me/desktop/)。

安装 [draw tools](https://static.iitc.me/build/release/plugins/draw-tools.user.js) 和 [Ingress IITC Portal Multi Export](https://github.com/modkin/Ingress-IITC-Multi-Export/raw/master/multi_export.user.js) 两个 IITC 插件。

Intel Map 的网址已经更新，将 draw tools 的 `UserScript` 段按如下修改。

```js
// ==UserScript==
// @id             iitc-plugin-draw-tools@breunigs
// @name           IITC plugin: draw tools
// @category       Layer
// @version        0.7.0.20170108.21732
// @namespace      https://github.com/jonatkins/ingress-intel-total-conversion
// @updateURL      https://static.iitc.me/build/release/plugins/draw-tools.meta.js
// @downloadURL    https://static.iitc.me/build/release/plugins/draw-tools.user.js
// @description    [iitc-2017-01-08-021732] Allow drawing things onto the current map so you may plan your next move.
// @match          http*://*intel.ingress.com/*
// @grant          none
// ==/UserScript==
```

使用 draw tools 绘制多边形，如图。

<img src="https://img-cdn.akass.cn/12/2022/08/6301c6dce0d21.png!wp">

选择右侧 Multi Export 中的 Polygon-Maxfield。

<img src="https://img-cdn.akass.cn/12/2022/08/6301c77c0c699.png!wp">

Select all 后复制到一个文本文档中。

<img src="https://img-cdn.akass.cn/12/2022/08/6301de4caf938.png!wp">

**对于 Windows 的特别提醒**：你可能需要将这个文档按 gbk 编码存储。

一个简单的做法是使用 VSCode，在右下角选择编码。

<img src="https://img-cdn.akass.cn/12/2022/08/6301e18557eb4.png!wp">

<img src="https://img-cdn.akass.cn/12/2022/08/6301e1b22a81b.png!wp">

<img src="https://img-cdn.akass.cn/12/2022/08/6301e1c8c4311.png!wp">

# 运行！

一个简单的例子：

```
python3 bin/maxfield-plan 1.txt -n 1 -c 0 -o test -v
```

解释：

- `1.txt` Portal 信息文件；
- `-n 1` 单人完成；
- `-c 0` 使用全核心计算；
- `-o test` 输出到 `test` 文件夹；
- `-v` 详细信息。

全部参数可以通过 `python3 bin/maxfield-plan -h` 获得帮助。

跑得挺慢的，我的 E5-2680 v4 $\times$ 2 计算一个 28 Portals 大约花费 14s。而完成全部图片生成后总计花费约 50s。

因此你可以通过参数 `--skip_step_plots` 取消每步的图片生成。

# 出门！

至此你已经获得了拉完美 Field 所需的全部资料。

- `frames` 文件夹中存放着每一步的参考；
- `agent_*_assignment.txt` 写明了每位参与特工的 link 分配；
- `agent_assignments.txt` 写明了全部的 link 分配；
- `agent_key_preparation.txt` 写明了每位参与特工所需准备的 key；
- `key_preparation.txt` 写明了全部所需准备的 key；
- `link_map.png` 是完成后的参考图；
- `ownership_preparation.txt` 写明了需要提前插满脚的 Portal；

我推荐将 `ownership_preparation.txt` 、 `key_preparation.txt` 和 `agent_assignments.txt` 打印出来。

当你按照以上资料完全准备后，直接按照 `agent_assignments.txt` 拉 Link 即可。