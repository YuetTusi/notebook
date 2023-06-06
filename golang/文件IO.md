## 读


Go语言中文件读取最简单的方法是使用`os.ReadFile()`，将文件一次性读取：

```go
    chunk, err := os.ReadFile("./resource/demo.txt")
    if err != nil {
        panic(err)
    }
    fmt.Println(chunk) //[228 184 173 232 182 133 ...
    fmt.Println(string(chunk)) //中超第9轮最佳球员候选：艾克森...
```

ReadFile方法返回整个文件的字节切片`[]byte`，这种方法不适合读取大文件。

> 在go1.16版本后，ioutil包已废弃，已将此包的功能移到os包和io包中

也可以自定义读取长度以适应大文件：

```go
    file, err := os.Open("./resource/demo.txt")

    if err != nil {
        panic(err)
    }
    
    defer file.Close() //关闭释放文件

    reader := bufio.NewReader(file) //创建带缓中区的reader
    var data []byte = make([]byte, 64)

    for {
        chunk := make([]byte, 64)
        _, err := reader.Read(chunk) //一次读取64字节

        if err == io.EOF {
            fmt.Println("读取结束")
            break
        } else {
            data = append(data, chunk...) //将每次读取的内容追加到data中
        }
    }
    fmt.Println(string(chunk)) //中超第9轮最佳球员候选：艾克森...
```

`bufio`包实现了缓冲的IO操作，通常我们进行文件读写时使用它非常方便。

### 读取JSON文件

方法为将JSON文件先读到内存中，使用`json.Unmarshal()`，将JSON文本内容转为对应的结构体。

现有JSON内容：
```json
[
    {
        "name": "应用",
        "level": 0,
        "children": [
            { "name": "微信", "level": 1 },
            { "name": "QQ", "level": 1 }
        ]
    }
]
```

针对JSON格式定义结构体：
```go
type TreeNode struct {
    Name     string     `json:"name"`
    Level    float64    `json:"level"`
    Children []TreeNode `json:"children,omitempty"`
}
```

结构体字段右侧的反引号为结构体标签，可定义字段转换JSON时的处理方式。
如：`json:name`，表示字段名为小写name，`omitempty`表示当字段为空值时不转换

读取文件并转换为struts：

```go
func ReadJson(name string, jsonType any) error {

    file, err := os.Open(name)

    if err != nil {
        return err
    }
    defer file.Close()
    total := 0 //记录读取字节数
    data := make([]byte, 0)

    for {
        chunk := make([]byte, 64)
        n, err := file.Read(chunk) //一次读取64byte
        if err == io.EOF {
            break
        } else {
            total += n //累计字节数
            data = append(data, chunk...)
        }
    }

    jsonErr := json.Unmarshal(data[:total], jsonType)
    return jsonErr
}
```

调用`ReadJson()`：
```go
func main() {

    tree := make([]TreeNode, 0)
    err := ReadJson("./resource/tree.json", &tree)

    if err != nil {
        panic(err)
    }

    fmt.Println(tree[0])
}
```


## 写

## 常用文件操作

### 读取文件状态

```go
    stat, err := os.Stat("./resource/demo.txt")
    if err != nil {
        panic(err)
    }

    fmt.Printf("文件名：%v\n", stat.Name())
    fmt.Printf("是目录：%v\n", stat.IsDir())
    fmt.Printf("文件大小：%v\n", stat.Size())
    fmt.Printf("修改时间：%v\n", stat.ModTime())
```

输出：
```txt
文件名：demo.txt
是目录：false
文件大小：818
修改时间：2023-06-01 11:14:48.1521881 +0800 CST
```

### 判断文件存在

借助`os.Stat()`方法，如果返回的错误为`nil`，则表示文件存在。如果有错误，将error传入`os.IsExist()`再进一步判断：

```go
func IsExist(name string) bool {
    _, err := os.Stat(name)
    if err != nil {
        return os.IsExist(err) //根据error对象进一步判断
    } else {
        return true //没有error，则表示文件直接存在
    }
}
```

### 读取目录

读取目录使用`os.ReadDir()`方法：

```go
    dir, err := os.ReadDir("./")

    if err != nil {
        panic(err)
    }

    for _, v := range dir {
        fmt.Printf("名称：%v,是否是目录：%v\n", v.Name(), v.IsDir())
    }
```

输出：
```txt
名称：file.go,是否是目录：false
名称：go.mod,是否是目录：false
名称：main.go,是否是目录：false
名称：resource,是否是目录：true
```

### 创建文件/目录

创建目录使用`os.Mkdir()`方法

```go
err := os.Mkdir("./resource/test", 0666)

if err != nil {
	fmt.Println("创建失败")
} else {
	fmt.Println("创建成功")
}
```

`Mkdir()`方法只可在传入父级目录中创建1层目录（如果创建多级目录，使用`os.MkdirAll()`），当传入的父目录不存在，或创建的目录已存在时，都会报错。

参数2为创建目录的Unix权限，为3位8进制的数值，分别对应：所有者，用户组，其他。

|可读|可写|可执行|
|---|---|---|
|4|2|1|

如：创建一个目录test，所有用户、组、其他都为可读可写：`os.Mkdir("test", 0666)`。

如：创建一个目录test，仅当前用户可读写，其他只读：`os.Mkdir("test", 0644)`


### 删除文件/目录

删除使用`os.Remove()`当目录中存在文件则会失败，如果想一次性删除非空目录，使用`os.RemoveAll()`

```go
err := os.Remove("./resource/test")

if err != nil {
	fmt.Println("删除失败")
} else {
	fmt.Println("删除成功")
}
```