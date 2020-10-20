---
title: "Golang Xmltag"
date: 2020-10-20T10:47:58+08:00
tags: ["golang", "encode/xml"]
categories: ["编程语言"]
draft: false
---

# Golang Tag Marshal and UnMarshal

Golang 支持tag功能，对于固定结构的结构体非常友好，最近再尝试用golang重新编写netconf api,涉及到xml的生成与解析功能，着实研究了一下，作为总结。

如果想要了解一个api功能，最好的方式还是尝试查看官网资料，有幸国人已经有了翻译版本：

> https://studygolang.com/pkgdoc 

抄袭官方文档，理解官方文档,OYe!
## 定义结构体，增加xml tag功能

### Marshal example
Example
``` golang
type Address struct {
    City, State string
}
type Person struct {
    XMLName   xml.Name `xml:"person"`
    Id        int      `xml:"id,attr"`
    FirstName string   `xml:"name>first"`
    LastName  string   `xml:"name>last"`
    Age       int      `xml:"age"`
    Height    float32  `xml:"height,omitempty"`
    Married   bool
    Address
    Comment string `xml:",comment"`
}
v := &Person{Id: 13, FirstName: "John", LastName: "Doe", Age: 42}
v.Comment = " Need more details. "
v.Address = Address{"Hanga Roa", "Easter Island"}
output, err := xml.MarshalIndent(v, "  ", "    ")
if err != nil {
    fmt.Printf("error: %v\n", err)
}
os.Stdout.Write(output)
```
Output
```
  <person id="13">
      <name>
          <first>John</first>
          <last>Doe</last>
      </name>
      <age>42</age>
      <Married>false</Married>
      <City>Hanga Roa</City>
      <State>Easter Island</State>
      <!-- Need more details. -->
  </person>
```
- XMLName字段，如上所述，会省略
- 具有标签"-"的字段会省略
- 具有标签"name,attr"的字段会成为该XML元素的名为name的属性
- 具有标签",attr"的字段会成为该XML元素的名为字段名的属性
- 具有标签",chardata"的字段会作为字符数据写入，而非XML元素
- 具有标签",innerxml"的字段会原样写入，而不会经过正常的序列化过程
- 具有标签",comment"的字段作为XML注释写入，而不经过正常的序列化过程，该字段内不能有"--"字符串
- 标签中包含"omitempty"选项的字段如果为空值会省略
  空值为false、0、nil指针、nil接口、长度为0的数组、切片、映射
- 匿名字段（其标签无效）会被处理为其字段是外层结构体的字段

### UnMarshal example

Example
```golang

type Email struct {
    Where string `xml:"where,attr"`
    Addr  string
}
type Address struct {
    City, State string
}
type Result struct {
    XMLName xml.Name `xml:"Person"`
    Name    string   `xml:"FullName"`
    Phone   string
    Email   []Email
    Groups  []string `xml:"Group>Value"`
    Address
}
v := Result{Name: "none", Phone: "none"}
data := `
		<Person>
			<FullName>Grace R. Emlin</FullName>
			<Company>Example Inc.</Company>
			<Email where="home">
				<Addr>gre@example.com</Addr>
			</Email>
			<Email where='work'>
				<Addr>gre@work.com</Addr>
			</Email>
			<Group>
				<Value>Friends</Value>
				<Value>Squash</Value>
			</Group>
			<City>Hanga Roa</City>
			<State>Easter Island</State>
		</Person>
	`
err := xml.Unmarshal([]byte(data), &v)
if err != nil {
    fmt.Printf("error: %v", err)
    return
}
fmt.Printf("XMLName: %#v\n", v.XMLName)
fmt.Printf("Name: %q\n", v.Name)
fmt.Printf("Phone: %q\n", v.Phone)
fmt.Printf("Email: %v\n", v.Email)
fmt.Printf("Groups: %v\n", v.Groups)
fmt.Printf("Address: %v\n", v.Address)
```
Output

```
XMLName: xml.Name{Space:"", Local:"Person"}
Name: "Grace R. Emlin"
Phone: "none"
Email: [{home gre@example.com} {work gre@work.com}]
Groups: [Friends Squash]
Address: {Hanga Roa Easter Island}
```

- 如果结构体字段的类型为字符串或者[]byte，且标签为",innerxml"，
  Unmarshal函数直接将对应原始XML文本写入该字段，其余规则仍适用。
- 如果结构体字段类型为xml.Name且名为XMLName，Unmarshal会将元素名写入该字段
- 如果字段XMLName的标签的格式为"name"或"namespace-URL name"，
  XML元素必须有给定的名字（以及可选的名字空间），否则Unmarshal会返回错误。
- 如果XML元素的属性的名字匹配某个标签",attr"为字段的字段名，或者匹配某个标签为"name,attr"
  的字段的标签名，Unmarshal会将该属性的值写入该字段。
- 如果XML元素包含字符数据，该数据会存入结构体中第一个具有标签",chardata"的字段中，
  该字段可以是字符串类型或者[]byte类型。如果没有这样的字段，字符数据会丢弃。
- 如果XML元素包含注释，该数据会存入结构体中第一个具有标签",comment"的字段中，
  该字段可以是字符串类型或者[]byte类型。如果没有这样的字段，字符数据会丢弃。
- 如果XML元素包含一个子元素，其名称匹配格式为"a"或"a>b>c"的标签的前缀，反序列化会深入
  XML结构中寻找具有指定名称的元素，并将最后端的元素映射到该标签所在的结构体字段。
  以">"开始的标签等价于以字段名开始并紧跟着">" 的标签。
- 如果XML元素包含一个子元素，其名称匹配某个结构体类型字段的XMLName字段的标签名，
  且该结构体字段本身没有显式指定标签名，Unmarshal会将该元素映射到该字段。
- 如果XML元素的包含一个子元素，其名称匹配够格结构体字段的字段名，且该字段没有任何模式选项
  （",attr"、",chardata"等），Unmarshal会将该元素映射到该字段。
- 如果XML元素包含的某个子元素不匹配以上任一条，而存在某个字段其标签为",any"，
  Unmarshal会将该元素映射到该字段。
- 匿名字段被处理为其字段好像位于外层结构体中一样。
- 标签为"-"的结构体字段永不会被反序列化填写。

## 实现UnmarshalXML方法

1. 需求:实现一个如下形式的解析和xml生成
bgp下可能对应多个不同类型的子xml节点，其中Familys和F1s的结构体是固定的，但是BGP tag下存在多少个不通的类型不确定
```xml
<BGP>
    <Familys>
        <Family>
            <Name>Name</Name>
            <VRF>VRF</VRF>
            <Balance>
                <MaxBalance>4</MaxBalance>
                <MinBalance>0</MinBalance>
            </Balance>
        </Family>
        <Family>
            <Name>Name</Name>
            <VRF>VRF</VRF>
            <Balance>
                <MaxBalance>4</MaxBalance>
                <MinBalance>0</MinBalance>
            </Balance>
        </Family>
    </Familys>
    <F1s>
	<F1>
            <f1>1</f1>
            <f2>2</f2>
	</F1>
    </F1s>
</BGP>
```

```
import (
	"encoding/xml"
	"fmt"
)

// BGP define
type BGP struct {
	XMLName xml.Name `xml:"BGP"`
	Bgp     []interface{}
}

// BGPFamilys define
type BGPFamilys struct {
	XMLName   xml.Name    `xml:"Familys"`
	BGPFamily []BGPFamily `xml:"Family"` //必须填写, 与BGPFamily的xml tag对应，否则解析不出来，此处如果填写 BGPFamily可以不定义XMLName但是建议定义，如果单独生成BGPFamily时需要使用
}

// BGPFamily define
type BGPFamily struct {
	XMLName    xml.Name `xml:"Family"`
	Name       string   `xml:"Name"`
	VRF        string   `xml:"VRF"`
	MaxBalance int      `xml:"Balance>MaxBalance"` //在嵌套标签 <Balance><MaxBalance>0</MaxBalance></Balance>
	MinBalance int      `xml:"Balance>MinBalance"`
}

// 实现UnmarshalXML方法, 正确解析xml到GBP结构体中
func (b *BGP) UnmarshalXML(d *xml.Decoder, start xml.StartElement) error {
	// 当调用到UnmarshalXML时，start.Name 就是BGP这个tag
	b.XMLName = start.Name
	// grab any other attrs
	// decode inner elements
	for {
		t, err := d.Token() // 向下找到子元素
		if err != nil {
			return err
		}
		var i interface{}
		switch tt := t.(type) {
		case xml.StartElement: //判断是起始标签
			switch tt.Name.Local {
			case "Familys":
				i = new(BGPFamilys) // 
			}
			if i != nil {
				err = d.DecodeElement(i, &tt) //DecodeElement函数，从tt这个起始标签向下解析出i所代表的结构体
				if err != nil {
					return err
				}
				b.Bgp = append(b.Bgp, i)
				i = nil
			}
		case xml.EndElement://直到解析到</BGP>这个标签退出,判断是结束标签
			if tt == start.End() {
				return nil
			}
		}
	}
	//return nil
}
```
- DecodeElement 函数适用于半自动化解析结构体，直接通过UnMarshal解析表示全自动解析，通过Deocode一点点调用Token()函数被称为手动的话，那么DecodeElement就被称为半自动化

**根据官方资料,由于xml struct tag形式对字段顺序有着强要求，所以如果是自定义的化建议还是使用json**


