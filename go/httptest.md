[原文地址](https://studygolang.com/articles/10392?fr=sidebar)

# web开发 Handler测试利器httptest

我们用go开发一个Web Server后，打算单元测试写的handler函数，在不知道httptest之前，使用比较笨的方法就是编译运行该Web Server后，再用go编写一个客户端程序向该Web Server对应的route发送数据然后解析返回的数据。这个方法测试时非常麻烦，使用httptest来测试的话就非常简单，可以和testing测试一起使用。

## httptest基本使用方法

假设在server中handler已经写好

````go
http.HandleFunc("/health-check", HealthCheckHandler)

func HealthCheckHandler(w http.ResponseWriter, r *http.Request) {
    // A very simple health check.
    w.WriteHeader(http.StatusOK)
    w.Header().Set("Content-Type", "application/json")

    // In the future we could report back on the status of our DB, or our cache 
    // (e.g. Redis) by performing a simple PING, and include them in the response.
    io.WriteString(w, `{"alive": true}`)
}
````

测试如下：

````go
import (
    "net/http"
    "net/http/httptest"
    "testing"
)

func TestHealthCheckHandler(t *testing.T) {
    //创建一个请求
    req, err := http.NewRequest("GET", "/health-check", nil)
    if err != nil {
        t.Fatal(err)
    }

    // 我们创建一个 ResponseRecorder (which satisfies http.ResponseWriter)来记录响应
    rr := httptest.NewRecorder()

    //直接使用HealthCheckHandler，传入参数rr,req
    HealthCheckHandler(rr, req)

    // 检测返回的状态码
    if status := rr.Code; status != http.StatusOK {
        t.Errorf("handler returned wrong status code: got %v want %v",
            status, http.StatusOK)
    }

    // 检测返回的数据
    expected := `{"alive": true}`
    if rr.Body.String() != expected {
        t.Errorf("handler returned unexpected body: got %v want %v",
            rr.Body.String(), expected)
    }
}
````

然后就可以使用run test来执行测试.

如果Web Server有操作数据库的行为，需要在init函数中进行数据库的连接。

参考官方文档中的样例编写的另外一个测试代码：

````go
func TestHealthCheckHandler2(t *testing.T) {
    reqData := struct {
        Info string `json:"info"`
    }{Info: "P123451"}

    reqBody, _ := json.Marshal(reqData)
    fmt.Println("input:", string(reqBody))
    req := httptest.NewRequest(
        http.MethodPost,
        "/health-check",
        bytes.NewReader(reqBody),
    )

    req.Header.Set("userid", "wdt")
    req.Header.Set("commpay", "brk")

    rr := httptest.NewRecorder()
    HealthCheckHandler(rr, req)

    result := rr.Result()

    body, _ := ioutil.ReadAll(result.Body)
    fmt.Println(string(body))

    if result.StatusCode != http.StatusOK {
        t.Errorf("expected status 200,",result.StatusCode)
    }
}
````

注意不同的地方

- http.NewRequest替换为httptest.NewRequest。

- httptest.NewRequest的第三个参数可以用来传递body数据，必须实现io.Reader接口。

- httptest.NewRequest不会返回error，无需进行err!=nil检查。

解析响应时没直接使用ResponseRecorder，而是调用了Result函数。

## 参考
httptest doc
Testing Your (HTTP) Handlers in Go

本文来自：简书

感谢作者：kingeasternsun

查看原文：golang web开发 Handler测试利器httptest
