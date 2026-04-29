**sphinx_Loongsonlab_theme V1.0**

# 为企业文档标准化而构建的sphinx主题

# 项目介绍

Sphinx LoongsonLab Theme是一个为企业文档标准化而构建的sphinx主题，支持html，word和pdf标准渲染输出。

**项目亮点**:

- 简化了文档构建过程,用户只需关注文档内容而无需关心复杂配置
- 提供了美观、符合企业标准的HTML、PDF和Word渲染输出样式
- 针对PDF输出优化了LaTeX模板和封面样式,美化了PDF渲染效果
- 嵌入了适合打印美观的免费商用中文(SourceHanSansSC)和英文字体(DejaVuSans)
- 新增了Word输出模板和构建命令,实现Word输出与PDF保持一致样式
- 引入了常用的Sphinx扩展作为默认配置,减少用户配置工作
- 提供了evas命令快速初始化文档项目模板,make命令进行编译构建
- 所有配置参数均有中文注释说明,降低了使用门槛

本主题是基于sphinx_rtd_theme作为基础主题构建，做出了如下修改：

- 新增latex导言和封面的模板，美化了PDF渲染样式
- 嵌入了适合打印美观的免费商用中英文字体
- 简化了PDF构建过程，由make latexpdf命令更改为make pdf命令
- 新增word模板和构建命令，美化了word渲染样式，样式保持与PDF基本一致
- 新增make all命令，以及环境检查
- 简化conf.py配置过程，只需要更改文档内容信息即可
- 引入常用的sphinx扩展(默认)
- 在sphinx_evas_theme主题的基础上做了部分修改

用户通过简单的命令即可实现项目文档模板初始化(evas命令，取代sphinx-quickstart)、文档构建(make html, make docx, make pdf以及make all)工作。


## 项目参考

感谢下列开源项目为本项目提供支持:

[sphinx](https://github.com/sphinx-doc/sphinx)：开源文档构建工具

[sphinx_rtd_theme](https://github.com/readthedocs/sphinx_rtd_theme)：使用最为广泛的Sphinx主题

[sphinx_idf_theme](https://github.com/espressif/sphinx_idf_theme)：乐鑫科技的Sphinx主题

[sphinx-jupyterbook-latex](https://github.com/executablebooks/sphinx-jupyterbook-latex)：支持LaTeX 输出的Sphinx扩展

[esp-docs](https://github.com/espressif/esp-docs)：乐鑫科技基于Sphinx的开源文档构建环境

[sphinx_evas_theme](https://github.com/ikiwihome/sphinx_evas_theme)：Sphinx EVAS Theme Sphinx的开源文档构建环境

