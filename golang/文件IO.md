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

> 在go1.16版本后，ioutil包已废弃，已将此包的功能移到os和io包中


也可以自定义读取长度以适应大文件：

```go
    file, err := os.Open("./resource/demo.txt")

    if err != nil {
        panic(err)
    }
    
    defer file.Close()

    reader := bufio.NewReader(file)
    var data []byte = make([]byte, 0)

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