## 1- 创建urp项目

1. 创建urp项目

2. remove readme

3. 检查是否是urp：

   1. graphics窗口对应的是 universalranderpipeline

      ![image-20230311231834327](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/11/image-20230311231834327-9c44a9.png)

   2. quality是highquality

      ![image-20230311231751609](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/11/image-20230311231751609-6febfa.png)

      删除上面两个：

      ![image-20230311232050855](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/11/image-20230311232050855-8f3c37.png)

   3. 删除setting文件夹下设置，仅留下highfidelity

      ![image-20230311232227202](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/11/image-20230311232227202-732d62.png)



## visual setup

![image-20230311232654870](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/11/image-20230311232654870-0cbce6.png)

添加扩展（非必须）

![image-20230311232759559](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/11/image-20230311232759559-259101.png)



按自己的喜好来，这是笔者的配置

![image-20230311233737675](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/11/image-20230311233737675-f9d62f.png)

## 命名规范

- 花点时间决定一个合适的名字
- 不要害怕重新命名事物
- 不要使用单字母的名字
- 不要使用首字母缩写词

![image-20230311234011550](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/11/image-20230311234011550-f0d349.png)



## 导包

直接双击package文件全选导入就行

![image-20230311234556894](https://pic-1257412153.cos.ap-nanjing.myqcloud.com/images/images/2023/03/11/image-20230311234556894-6cfbcf.png)

## PostProcessing

重命名场景为：`GameScene`



点击`GlobalVolume`，在Inspector窗口中删除Volume的Profile并新创建一个

## Character Controller

将**视觉效果**和**逻辑效果**分离


```c#
    public static Player Instance { 
        get { return instanceField; }
        set { Instance = value; }
    }
    public static Player instanceField;
    public static Player GetInstanceField() {
        return instanceField; 
    }
    public static void SetInstanceField(Player instanceField) {
         Player.instanceField = instanceField;
    }
```

