### 字符串

- 数字转字符串姿势

  ```go
  //使用string()将数字转字符串时，结果是会是该数组所代表的ascii字符
  fmt.Println(string(97))		//a，不会打印"97"
  
  //正确做法应该是使用strconv.Itoa()函数
  fmt.Println(strconv.Itoa(97))
  //或者用fmt.Sprintf()，通常这种做法语义性更强，但效率没有前者高
  fmt.Println(fmt.Sprintf("%c", 97))
  ```

- 遍历字符串

  ```go
  str := "我abc123"
  for i := 0; i < len(str); i++ {
  	fmt.Printf("%d %c\n", i, str[i])
  }
  //得到以下输出
  //0 æ
  //1 
  //2 
  //3 a
  //4 b
  //5 c
  //6 1
  //7 2
  //8 3
  ```

  - 字符串底层是一个byte(字节)数组，Go 语言里的字符串的内部实现使用utf-8编码，在utf-8下，中文通常占3个byte(字节)，所以遍历字符串(遍历byte数组)，遍历到中文"我"（前三个字节），就乱码了。utf-8英文和数字都是一个字节，不会乱码。

  - 可以将字符串转换为rune数组(byte数组转rune数组)。

    ```go
    str := "我abc123"
    for i := 0; i < len([]rune(str)); i++ {
    	fmt.Printf("%d %c\n", i, []rune(str)[i])
    }
    //正常展示
    //0 我
    //1 a
    //2 b
    //3 c
    //4 1
    //5 2
    //6 3
    ```

  - rune类型，代表一个 utf-8字符。byte 型，代表了ASCII码的一个字符。

    ```go
    str := "我abc123"
    fmt.Println([]byte(str))   //[230 136 145 97 98 99 49 50 51]
    fmt.Println([]rune(str))   //[25105 97 98 99 49 50 51]
    fmt.Printf("%c", 25105)	   //我
    ```

  - 或者使用range，range循环在遍历字符串的过程中，会对字符串进行 utf-8 解码

    ```go
    str := "我abc123"
    for k, v := range str {
    	fmt.Printf("%d %c\n", k, v)
    }
    //结果如下
    //0 我
    //3 a
    //4 b
    //5 c
    //6 1
    //7 2
    //8 3
    ```

- 字符串是不可变的，要修改他，得转换为[]byte()或[]rune，通过下标索引修改

### 结构体

- 结构体构造函数可变入参

  ```go
  type User struct {
  	Id   int
  	Name string
  	Age  int
  }
  
  type SetUserFunc func(*User)
  
  func NewUser(funs ...SetUserFunc) *User {
  	u := new(User)
  	for _, f := range funs {
  		f(u)
  	}
  	return u
  }
  
  func SetUserId(id int) SetUserFunc {
  	return func(u *User) {
  		u.Id = id
  	}
  }
  
  func SetUserName(name string) SetUserFunc {
  	return func(u *User) {
  		u.Name = name
  	}
  }
  
  func main() {
  	u := NewUser(
  		SetUserId(10),
  		SetUserName("小明"),
  	)
  	fmt.Println(u)
  }
  ```

  
