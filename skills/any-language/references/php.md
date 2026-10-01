# PHP

Source: https://bugsradar.com/use-cases/any-language/

```php
$ch = curl_init('https://api.bugsradar.com/api/v3/notify?level=error&category=orders');
curl_setopt_array($ch, [
    CURLOPT_POST => true,
    CURLOPT_POSTFIELDS => "Order $orderId failed: " . $e->getMessage(),
    CURLOPT_HTTPHEADER => ['X-Api-Key: ' . getenv('BUGSRADAR_KEY'), 'Content-Type: text/plain; charset=utf-8'],
    CURLOPT_TIMEOUT => 15,
    CURLOPT_RETURNTRANSFER => true,
]);
curl_exec($ch);
curl_close($ch);
```
