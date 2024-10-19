회사에서 벌크 API개발을 맡게 되었는데 이미 틀이 잡혀있어서 어렵지 않았다.
엘라스틱 서치에서도 벌크를 사용한 개발이 가능하다는데 RestHighLevelClient 를 사용하여 사용한다고 한다.(gpt4)

``` text
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.10.0</version>
</dependency>
```

```java
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.bulk.BulkResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.xcontent.XContentType;

import java.io.IOException;

public class BulkApiExample {

    private final RestHighLevelClient client;

    public BulkApiExample(RestHighLevelClient client) {
        this.client = client;
    }

    public void performBulkOperation() throws IOException {
        // 1. BulkRequest 객체 생성
        BulkRequest bulkRequest = new BulkRequest();

        // 2. 여러 IndexRequest 추가
        bulkRequest.add(new IndexRequest("index_name")
                .id("1")
                .source("{\"field\":\"value1\"}", XContentType.JSON));
        bulkRequest.add(new IndexRequest("index_name")
                .id("2")
                .source("{\"field\":\"value2\"}", XContentType.JSON));
        // 필요한 만큼 요청 추가

        // 3. Bulk 요청 실행
        BulkResponse bulkResponse = client.bulk(bulkRequest, RequestOptions.DEFAULT);

        // 4. 응답 처리
        if (bulkResponse.hasFailures()) {
            System.out.println("Some requests failed: " + bulkResponse.buildFailureMessage());
        } else {
            System.out.println("All requests successful");
        }
    }
}
```
