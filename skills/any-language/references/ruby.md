# Ruby

The same request as the Go and PHP examples on https://bugsradar.com/use-cases/any-language/, with Net::HTTP:

```ruby
require 'net/http'
require 'uri'

def notify_bugsradar(text, level: 'error', category: nil)
  query = URI.encode_www_form({ level: level, category: category }.compact)
  uri = URI("https://api.bugsradar.com/api/v3/notify?#{query}")
  request = Net::HTTP::Post.new(uri)
  request['X-Api-Key'] = ENV.fetch('BUGSRADAR_KEY')
  request['Content-Type'] = 'text/plain; charset=utf-8'
  request.body = text
  Net::HTTP.start(uri.host, uri.port, use_ssl: true, open_timeout: 5, read_timeout: 15) do |http|
    http.request(request)
  end
rescue StandardError
  nil # an alert must never break the code that sends it
end

notify_bugsradar("Order #{order_id} failed: #{error.message}", category: 'orders')
```
