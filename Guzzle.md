# Bắt đầu nhanh — Tài liệu Guzzle

Đây là phần giới thiệu nhanh về Guzzle kèm những ví dụ.
Nếu bạn chưa cài đặt Guzzle, hãy đến trang cài đặt

## Tạo một request

Bạn có thể gửi yêu cầu với Guzzle bằng cách sử dụng đối tượng `GuzzleHttpClientInterface`.

### Tạo một client
    
    
    sử dụng GuzzleHttpClient;
    
    $client = new Client([
        // URI cơ bản được sử dụng với các yêu cầu tương đối
        'base_uri' => 'http://httpbin.org',
        // Bạn có thể đặt ra số lượng những lựa chọn yêu cầu còn thiếu.
        'timeout'  => 2.0,
    ]);
    

Clients không thể thay đổi trong Guzzle 6, nghĩa là bạn không thể thay đổi phần còn thiếu được sử dụng bởi một client sau khi nó được tạo ra .

Người tạo client chấp nhận một dãy kết hợp các lựa chọn : 

`base_uri`
: 

(string|UriInterface) URI cơ bản của client được hợp nhất vào URIs tương đối Có thể là một chuỗi hoặc ví dụ của UriInterface. Khi một URI tương đốiđược cung cấp cho một client, client sẽ kết nối URI cơ bản với URI tương đối bằng cách sử dụng quy tắc được mô tả trong[RFC 3986, section 2][2].
    
    
    // Tạo một client với một URI cơ bản
    $client = new GuzzleHttpClient(['base_uri' => 'https://foo.com/api/']);
    // Gửi một request tới https://foo.com/api/test
    $response = $client->request('GET', 'test');
    // Gửi một request tới https://foo.com/root
    $response = $client->request('GET', '/root');
    

Đừng cảm thấy như việc đọc RFC 3986? Sau đây là một số ví dụ nhanh về việc một `base_uri` được giải quyết với một URI khác.

| base_uri              | URI              | Result                   |  
| --------------------- | ---------------- | ------------------------ |  
| `http://foo.com`      | `/bar`           | `http://foo.com/bar`     |  
| `http://foo.com/foo`  | `/bar`           | `http://foo.com/bar`     |  
| `http://foo.com/foo`  | `bar`            | `http://foo.com/bar`     |  
| `http://foo.com/foo/` | `bar`            | `http://foo.com/foo/bar` |  
| `http://foo.com`      | `http://baz.com` | `http://baz.com`         |  
| `http://foo.com/?bar` | `bar`            | `http://foo.com/bar`     |  

`handler`
: (trả trước) Chức năng chuyển các yêu cầu HTTP  over the wire. Chức năng đó có tên `Psr7HttpMessageRequestInterface` và một dãy những lựa chọn chuyển đổi, và phải trả về một`GuzzleHttpPromisePromiseInterface` mà được thực hiện bởi một `Psr7HttpMessageResponseInterface` on success. `handler` là sự lựa chọn công cụ xây dựng duy nhất mà không thể bị overridden trong mỗi lựa chọn yêu cầu

`...`
: (mixed) Tất cả các lựa chọn khác được thông qua làm công cụ xây dựng được sử dụng như các lựa chọn yêu cầu còn thiếu với mọi yêu cầu được tạo bởi client .
### Gửi Requests

Các phương pháp magic trong client giúp cho các yêu cầu đồng bộ được gửi đi một cách dễ dàng:
    
    
    $response = $client->get('http://httpbin.org/get');
    $response = $client->delete('http://httpbin.org/delete');
    $response = $client->head('http://httpbin.org/get');
    $response = $client->options('http://httpbin.org/get');
    $response = $client->patch('http://httpbin.org/patch');
    $response = $client->post('http://httpbin.org/post');
    $response = $client->put('http://httpbin.org/put');
    

Bạn có thể tạo một yêu cầu và gửi nó với client khi bạn sẵn sàng :
    
    
    sử dụng GuzzleHttpPsr7Request;
    
    $request = new Request('PUT', 'http://httpbin.org/put');
    $response = $client->send($request, ['timeout' => 2]);
    

Các đối tượng client cung cấp rất nhiều tính linh hoạt trong việc các yêu cầu được chuyển đổi như thế nào bao gồm các lựa chọn yêu cầu còn thiếu , default handler stack middleware được sử dụng bởi mỗi request, và một URI cơ bản cho phép bạn gửi yêu cầu với URIs tương đối.

Bạn có thể tìm hiểu nhiều hơn về client middleware trong trang [_Handlers and Middleware_][3] của tài liệu

### Yêu cầu đồng bộ

Bạn có thể gửi các yêu cầu đồng bộ bằng cách sử dụng những phương pháp magic được cung cấp bởi một client:
    
    
    $promise = $client->getAsync('http://httpbin.org/get');
    $promise = $client->deleteAsync('http://httpbin.org/delete');
    $promise = $client->headAsync('http://httpbin.org/get');
    $promise = $client->optionsAsync('http://httpbin.org/get');
    $promise = $client->patchAsync('http://httpbin.org/patch');
    $promise = $client->postAsync('http://httpbin.org/post');
    $promise = $client->putAsync('http://httpbin.org/put');
    

Bạn cũng có thể sử dụng phương pháp sendAsync() và requestAsync() của một client:
    
    
    sử dụng GuzzleHttpPsr7Request;
    
    // Tạo một đối tượng yêu cầu PSR-7 để gửi
    $headers = ['X-Foo' => 'Bar'];
    $body = 'Hello!';
    $request = new Request('HEAD', 'http://httpbin.org/head', $headers, $body);
    
    // Hoặc, nếu bạn không cần thông qua trong một ví dụ yêu cầu:
    $promise = $client->requestAsync('GET', 'http://httpbin.org/get');
    

Lời hứa được trả về bởi những phương pháp này [Promises/A+ spec][4], cung cấp bởi [Guzzle promises library][5]. Điều đó có nghĩa rằng bạn có thể chain `then()` calls off of the promise. Sau đó những sự kêu gọi này được thực hiện với thành công  `PsrHttpMessageResponseInterface` hoặc bị từ chối với sự loại bỏ.
    
    
    use PsrHttpMessageResponseInterface;
    use GuzzleHttpExceptionRequestException;
    
    $promise = $client->requestAsync('GET', 'http://httpbin.org/get');
    $promise->then(
        function (ResponseInterface $res) {
            echo $res->getStatusCode() . "n";
        },
        function (RequestException $e) {
            echo $e->getMessage() . "n";
            echo $e->getRequest()->getMethod();
        }
    );
    

### Gửi yêu cầu đồng thời

Bạn có thể sử dụng đồng thời các yêu cầu phức tạp bằng cách sử dụng lời hứa và yêu cầu đồng bộ
    
    use GuzzleHttpClient;
    use GuzzleHttpPromise;
    
    $client = new Client(['base_uri' => 'http://httpbin.org/']);
    
    // Bắt đầu mỗi yêu cầu nhưng không ngăn chặn
    $promises = [
        'image' => $client->getAsync('/image'),
        'png'   => $client->getAsync('/image/png'),
        'jpeg'  => $client->getAsync('/image/jpeg'),
        'webp'  => $client->getAsync('/image/webp')
    ];
    
    // Đợi tất cả yêu cầu hoàn thành. Bỏ đi một  ConnectException
    // nếu yêu cầu thất bại
    $results = Promiseunwrap($promises);
    
    //  Đợi tất cả yêu cầu hoàn thành, thậm chí nếu một trong số đó thất bại
    $results = Promisesettle($promises)->wait();
    
    // Bạn có thể truy cập mỗi kết quả bằng cách sử dụng được cung cấp 
    // function.
    echo $results['image']['value']->getHeader('Content-Length')[0]
    echo $results['png']['value']->getHeader('Content-Length')[0]
    

Bạn có thể sử dụng đối tượng `GuzzleHttpPool` khi bạn có một số lượng yêu cầu không xác định mà bạn muốn gửi.

    use GuzzleHttpPool;
    use GuzzleHttpClient;
    use GuzzleHttpPsr7Request;
    
    $client = new Client();
    
    $requests = function ($total) {
        $uri = 'http://127.0.0.1:8126/guzzle-server/perf';
        for ($i = 0; $i < $total; $i++) {
            yield new Request('GET', $uri);
        }
    };
    
    $pool = new Pool($client, $requests(100), [
        'concurrency' => 5,
        'fulfilled' => function ($response, $index) {
            // this is delivered each successful response
        },
        'rejected' => function ($reason, $index) {
            // this is delivered each failed request
        },
    ]);
    
    // Bắt đầu chuyển và tạo mới một promise
    $promise = $pool->promise();
    
    // Buộc nhóm yêu cầu phải hoàn thành
    $promise->wait();
    

Hoặc sử dụng một lệnh đống sẽ trả lại một promise từ nhóm yêu cầu đóng.
    
    
    $client = new Client();
    
    $requests = function ($total) use ($client) {
        $uri = 'http://127.0.0.1:8126/guzzle-server/perf';
        for ($i = 0; $i < $total; $i++) {
            yield function() use ($client, $uri) {
                return $client->getAsync($uri);
            };
        }
    };
    
    $pool = new Pool($client, $requests(100));
    

## Sử dụng phản hồi

Trong các ví dụ trước, chúng tôi đã khôi phục lại một biến `$response` hoặc chúng tôi đã gửi những phản hồi từ một promise. Đối tượng phản hồi thực hiện một phản hồi PSR-7, `PsrHttpMessageResponseInterface` , và chứa những thông tin hữu ích.

Bạn có thể lấy những trạng thái mã và chi tiết lý do của phản hồi:
    
    
    $code = $response->getStatusCode(); // 200
    $reason = $response->getReasonPhrase(); // OK
    

Bạn có thể lấy tiêu đề từ phản hồi:
    
    
    // Kiểm tra nếu tiêu đề đã tồn tại
    if ($response->hasHeader('Content-Length')) {
        echo "It exists";
    }
    
    // Lấy một tiêu đề từ phản hồi
    echo $response->getHeader('Content-Length');
    
    // Lấy tất cả những tiêu đề phản hồi
    foreach ($response->getHeaders() as $name => $values) {
        echo $name . ': ' . implode(', ', $values) . "rn";
    }
    

Nội dung phản hội có thể được khôi phục bằng cách sử dụng phương pháp`getBody` .TNội dung có thể được sử dụng như một dãy, cast to a string, hoặc như một stream like object.
    
    
    $body = $response->getBody();
    // Implicitly cast the body to a string and echo it
    echo $body;
    // Explicitly cast the body to a string
    $stringBody = (string) $body;
    // Read 10 bytes from the body
    $tenBytes = $body->read(10);
    // Read the remaining contents of the body as a string
    $remainingBytes = $body->getContents();
    

## Query String Parameters (Thông số chuỗi truy vấn)


Bạn có thể cung cấp thông số chuỗi truy vấn với một yêu cầu bằng nhiều cách.

Bạn có thể thiết lập thông số chuỗi truy vấn trong URI của request:
    
    $response = $client->request('GET', 'http://httpbin.org?foo=bar');
    

Bạn có thể ghi chú thông số chuỗi truy vấn bằng cách sử dụng lựa chọn yêu cầu`query` như một dãy.
    
    
    $client->request('GET', 'http://httpbin.org', [
        'query' => ['foo' => 'bar']
    ]);
    

Việc cung cấp các lựa chọn như một dãy sẽ sử dụng chức năng `http_build_query` của PHP để format chuỗi truy vấn.

Và cuối cùng, bạn có thể cung cấp lựa chọn yêu cầu `query` như một chuỗi
    
    
    $client->request('GET', 'http://httpbin.org', ['query' => 'foo=bar']);
    

## Uploading Data (Tải dữ liệu lên)

Guzzle cung cấp một số phương pháp để tải dữ liệu.

Bạn có thể gửi yêu cầu chứa một chuỗi dữ liệu bằng cách thông qua một chuỗi, nguồn được trả về từ `fopen`, hoặc một ví dụ của một  `PsrHttpMessageStreamInterface` đến `body` lựa chọn yêu cầu.
    
    
    // Cung cấp phần chính như một chuỗi.
    $r = $client->request('POST', 'http://httpbin.org/post', [
        'body' => 'raw data'
    ]);
    
    // Cung cấp một nguồn fopen.
    $body = fopen('/path/to/file', 'r');
    $r = $client->request('POST', 'http://httpbin.org/post', ['body' => $body]);
    
    // Sử dụng stream_for() function để tạo một dãy PSR-7.
    $body = GuzzleHttpPsr7stream_for('hello!');
    $r = $client->request('POST', 'http://httpbin.org/post', ['body' => $body]);
    

Một cách dễ dàng để tại lên dữ liệu JSON và set the appropriate header là sử dụng `json` request option:
    
    
    $r = $client->request('PUT', 'http://httpbin.org/put', [
        'json' => ['foo' => 'bar']
    ]);
    

### POST/Form Requests

Bên cạnh việc chỉ rõ raw data của một request sử dụng `body` rlựa chọn yêu cầu, Guzzle cung cấp abstractions có ích qua việc gửi dữ liệu POST.

#### Sending form fields

Việc gửi `application/x-www-form-urlencoded` POST requests yêu cầu bạn phải chỉ rõ POST fields như một dãy trong các lựa chọn request `form_params`.
    
    
    $response = $client->request('POST', 'http://httpbin.org/post', [
        'form_params' => [
            'field_name' => 'abc',
            'other_field' => '123',
            'nested_field' => [
                'nested' => 'hello'
            ]
        ]
    ]);
    

#### Gửi dạng files

Bạn có thể gửi files kèm một dạng (`multipart/form-data` POST requests), sử dụng lựa chọn yêu cầu`multipart` . `multipart` chấp nhận một dãy các dãy kết hợp, nơi mỗi dãy kết hợp chứa các key sau đây:

* tên: (required, string) lỗi dấn đến tên dạng file.
* nội dung: (required, mixed) Cung cấp một chuỗi để gửi files dưới dạng chuỗi,cũng cấp một nguồn fopen để stream tnội dung từ một PHP stream, ohoặc cung cấp một `PsrHttpMessageStreamInterface` để stream tnội dung từ một PSR-7 stream.
    
    
    $response = $client->request('POST', 'http://httpbin.org/post', [
        'multipart' => [
            [
                'name'     => 'field_name',
                'contents' => 'abc'
            ],
            [
                'name'     => 'file_name',
                'contents' => fopen('/path/to/file', 'r')
            ],
            [
                'name'     => 'other_file',
                'contents' => 'hello',
                'filename' => 'filename.txt',
                'headers'  => [
                    'X-Foo' => 'this is an extra header to include'
                ]
            ]
        ]
    ]);
    

## Cookies

Guzzle có thể duy trì cookie session cho bạn nếu được cung cấp sử dụng lựa chon request `cookies`. Khi gửi đi một request, lựa chọn `cookies` phải được thiết lập là ví dụ của `GuzzleHttpCookieCookieJarInterface`.
    
    
    // Use a specific cookie jar
    $jar = new GuzzleHttpCookieCookieJar;
    $r = $client->request('GET', 'http://httpbin.org/cookies', [
        'cookies' => $jar
    ]);
    

Bạn có thể đặt `cookies` đến `true` trong một công cụ xây dựng client nếu bạn muốn sử dụng cookie jar được chia sẻ cho tất cả requests.
    
    
    // Use a shared client cookie jar
    $client = new GuzzleHttpClient(['cookies' => true]);
    $r = $client->request('GET', 'http://httpbin.org/cookies');
    

## Redirects (Điều hướng)

Guzzle sẽ tự động theo dõi một địa chỉ mới nếu bạn không nói. Bạn có thể  tùy chỉnh redirect behavior bằng cách sử dụng`allow_redirects` request option.

* Set to `true` để cho phép những địa chỉ mới thông thường với con số tối đa là 5 địa chỉ mới. Đó là phần cài đặt bị thiếu.
* Set to `false` để ngăn chặn ác địa chỉ mới.
* Thông qua một tập hợp mảng chứa 'max' key chỉ ra các số lớn nhất điều hướng và tùy ý cung cấp một giá trị key 'strict' để chỉ ra có hay không việc sử dụng điều hướng nghiêm nghặt RFC  (Nghĩa là điều hướng các yêu cầu POST với các yêu cầu POST so với việc hầu hết các trình duyệt là điều hướng yêu cầu POST với các yêu cầu GET).
    
    
    $response = $client->request('GET', 'http://github.com');
    echo $response->getStatusCode();
    // 200
    

Các ví dụ sau đây thể hiện việc redirects có thể bị ngăn chặn.
    
    
    $response = $client->request('GET', 'http://github.com', [
        'allow_redirects' => false
    ]);
    echo $response->getStatusCode();
    // 301
    

## Exceptions (Ngoại lệ)

Guzzle ném các ngoại lệ từ các lỗi trong quá trình chuyển

* Trong trường hợp mạng lỗi mạng (hết thời gian kết nối, lỗi DNS, etc.), một `GuzzleHttpExceptionRequestException` sẽ bị bắt. Ngoại lệ này kéo dài từ `GuzzleHttpExceptionTransferException`. Các ngoại lệ này sẽ bắt bất kì ngoại lệ nào được quăng trong quá trình chuyển yêu cầu.
    
        use GuzzleHttpPsr7;
        use GuzzleHttpExceptionRequestException;
    
        try {
            $client->request('GET', 'https://github.com/_abc_123_404');
        }   catch (RequestException $e) {
            echo Psr7str($e->getRequest());
            if ($e->hasResponse()) {
                echo Psr7str($e->getResponse());
            }
        }
    

* Một ngoại lệ `GuzzleHttpExceptionConnectException`  sẽ ném trong sự kiện lỗi mạng. Ngoại lệ này kéo dài từ `GuzzleHttpExceptionRequestException`.
* Một `GuzzleHttpExceptionClientException` sẽ ném đến lỗi cấp 400 nếu `http_errors` gửi tùy chọn yêu cầu là true. Ngoại lệ này kéo dài từ `GuzzleHttpExceptionBadResponseException` và `GuzzleHttpExceptionBadResponseException` kéo dài từ `GuzzleHttpExceptionRequestException`.
    
        use GuzzleHttpExceptionClientException;
    
        try {
            $client->request('GET', 'https://github.com/_abc_123_404');
            }   catch (ClientException $e) {
            echo Psr7str($e->getRequest());
            Psr7str($e->getResponse());
         }
    

* Một `GuzzleHttpExceptionServerException` sẽ ném đến lỗi cấp 500 nếu `http_errors` gửi tùy chọn yêu cầu là true. Ngoại lệ này kéo dài từ `GuzzleHttpExceptionBadResponseException`.
* Một `GuzzleHttpExceptionTooManyRedirectsException` sẽ ném khi có nhiều điều hướng được theo dõi. Ngoại lệ này kéo dài từ `GuzzleHttpExceptionRequestException`.

Tất các các ngoại lệ trên được kéo dài từ `GuzzleHttpExceptionTransferException`.

## Environment Variables (Biến môi trường)

Guzzle cho thấy một vài biến môi trường có thể được sử dụng để tùy chỉnh động thái trong thư viện.

`GUZZLE_CURL_SELECT_TIMEOUT`
: Kiểm soát thời gian xử lý bằng giây curl_multi_* sẽ sử dụng khi bộ chọn tay sử dụng  `curl_multi_select()`. Một số hệ thống gặp vấn đề với việc thực thi PHP `curl_multi_select()` trong đó việc gọi hàm này luôn luôn dẫn đến thời gian chờ tối đa.

`HTTP_PROXY`

: Xác định proxy sử dụng khi gửi yêu cầu sử dụng giao thức "http"

Ghi chú: Bời vì biến HTTP_PROXY có thể chưa đầu vào người dùng tùy ý trên một số môi trường (CGI), biến chỉ được sử dụng trên CLI SAPI. Xem để biết thêm thông tin chi tiết.

`HTTPS_PROXY`
: Xác định proxy sử dụng khi gửi yêu cầu sử dụng giao thức "http"