# pycharm format code segment

pycharm使用autopep8的方法网上教程很多，内容都是一模一样的，这里不再赘述。仅列举下配置

```shell
Programs：autopep8 （Make sure you have installed）
Parameters: --in-place --aggressive --aggressive $FilePath$
Working directory: $ProjectFileDir$
Output filters：$FILE_PATH$\:$LINE$\:$COLUMN$\:.*
```

使用autopep8格式化整个文件是最优的选择，但是现实很骨感，如果整个项目之前并未通过工具进行代码格式化，那么你改了一行代码后，格式化整个文件并不一定是最好的选择，至少代码review时很痛苦。



当然最好的办法可能是将整个项目做一次格式化，之后大家都使用autopep8进行格式化就好了。但是遇到需要比较格式化前后的代码变化时，也很痛苦。

因此，折中的办法是把本次修改的代码、函数进行segment级别的格式化

## 配置方法

autopep8 --line-range指定要格式化的代码范围，只需在pycharm external tools中配置一下即可

1. copy前文的external tool，命名为auto-pep8-segment
2. Parameters修改为 ```--in-place --aggressive --aggressive $FilePath$ --line-range $SelectionStartLine$ $SelectionEndLine$``` 即在尾部增加--line-range参数

## 使用方法

1. 选择要格式化的代码
2. Right click -> external tools -> autopep8-segment