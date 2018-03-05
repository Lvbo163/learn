# 用mock server模拟调用(httptest)

mock是个好东东，在大项目或大公司，很实用，因为很多环境不是随时在开发环境可得的。

````go
package main
 
import (
    "testing"
    "net/http"
    "fmt"
    "net/http/httptest"
)
 
const checkMark = " OK! "
const ballotX = " ERROR! "
 
var feed = `<?xml version="1.0" encoding="UTF-8"?>
        <rss>
            <channel>
                <title>Going Go Programming</title>
                <description>Golang : https://github.com/goinggo</description>
                <link>http://www.goinggo.net/</link>
                <item>
                    <pubDate>Sun, 15 Mar 2015 15:04:00 +0000</pubDate>
                    <title>Object Oriented Programming Mechanics</title>
                    <description>Go is an object oriented language.</description>
                    <link>http://www.goinggo.net/2015/03/object-oriented</link>
                </item>
        </channel>
    </rss>`
 
func mockServer() *httptest.Server {
    f := func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(200)
        w.Header().Set("Content-Type", "application/xml")
        fmt.Fprintln(w, feed)
    }
    return httptest.NewServer(http.HandlerFunc(f))
}
     
func TestDownload(t *testing.T) {
    statusCode := http.StatusOK
 
    server := mockServer()
    defer server.Close()
     
    t.Log("Given the need to test downloading content.")
    {
        t.Logf("\tWhen checking \"%s\" for status code \"%d\"", server.URL, statusCode)
        {
            resp, err := http.Get(server.URL)
            if err != nil {
                t.Fatal("\tShould be able to make the Get call.", ballotX, err)
            }
            t.Log("\t\tShould be able to make the Get call.", checkMark)
            defer resp.Body.Close()
             
            if resp.StatusCode == statusCode {
                t.Logf("\t\tShould receive a \"%d\" status, %v", statusCode, checkMark)
            } else {
                t.Errorf("\t\tShould receive a \"%d\" status. %v %v", statusCode, ballotX, resp.StatusCode)
            }
        }
    }
}
````            
