### pip的导出和安装

把一个环境中的所有pip的包全部导出到一个文件中

```shell
pip freeze > requirements.txt
```

从指定的pip包配置文件安装pip包

```shell
pip install -r requirements.txt
```

