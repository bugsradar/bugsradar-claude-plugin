# Go

Source: https://bugsradar.com/use-cases/any-language/

```go
req, _ := http.NewRequest("POST",
    "https://api.bugsradar.com/api/v3/notify?level=error&category=orders",
    strings.NewReader("Order "+orderID+" failed: "+err.Error()))
req.Header.Set("X-Api-Key", os.Getenv("BUGSRADAR_KEY"))
req.Header.Set("Content-Type", "text/plain; charset=utf-8")
resp, sendErr := http.DefaultClient.Do(req)
if sendErr == nil {
    resp.Body.Close()
}
```

Use a client with a timeout instead of `http.DefaultClient` in real code, for example `&http.Client{Timeout: 15 * time.Second}`.
