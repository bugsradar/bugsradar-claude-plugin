# Java

The same request as the Go and PHP examples on https://bugsradar.com/use-cases/any-language/, with `java.net.http.HttpClient` (Java 11 and later):

```java
import java.net.URI;
import java.net.URLEncoder;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.charset.StandardCharsets;
import java.time.Duration;

public final class BugsRadarNotifier {
    private static final HttpClient CLIENT = HttpClient.newBuilder()
            .connectTimeout(Duration.ofSeconds(5))
            .build();

    public static void notify(String text, String level, String category) {
        try {
            String url = "https://api.bugsradar.com/api/v3/notify?level="
                    + URLEncoder.encode(level, StandardCharsets.UTF_8)
                    + "&category=" + URLEncoder.encode(category, StandardCharsets.UTF_8);
            HttpRequest request = HttpRequest.newBuilder(URI.create(url))
                    .timeout(Duration.ofSeconds(15))
                    .header("X-Api-Key", System.getenv("BUGSRADAR_KEY"))
                    .header("Content-Type", "text/plain; charset=utf-8")
                    .POST(HttpRequest.BodyPublishers.ofString(text, StandardCharsets.UTF_8))
                    .build();
            CLIENT.sendAsync(request, HttpResponse.BodyHandlers.discarding());
        } catch (Exception ignored) {
            // an alert must never break the code that sends it
        }
    }
}
```

Usage: `BugsRadarNotifier.notify("Order " + orderId + " failed: " + e.getMessage(), "error", "orders");`
