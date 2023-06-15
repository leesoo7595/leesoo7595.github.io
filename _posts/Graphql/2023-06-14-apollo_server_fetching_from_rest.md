---
layout: post
title: "Apollo Server - Fetching from REST"
date: 2023-06-14
tags: [Graphql]
comments: true
---

## Fetching from REST

RESTDataSource를 활용한 REST API에서 데이터 가져오기

`RESTDataSource` 클래스는 리졸빙할 때, REST APIs에서 데이터를 가져오는 것을 간단히 하고, 캐싱, 요청 중복 제거 및 에러를 처리하는 것에 도움을 준다.

<img width="882" alt="스크린샷 2023-06-14 오후 11 07 24" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/fc7011a7-5233-488d-a76e-4031f6fd6051">

### Creating subclasses

```
npm install @apollo/datasource-rest
```

서버에서는 각 REST API가 상호작용하여야하는 `RESTDataSource`에 대한 분리된 서브클래스를 정의하여야한다.

```typescript
import { RESTDataSource } from "@apollo/datasource-rest";

class MoviesAPI extends RESTDataSource {
  override baseURL = "https://movies-api.example.com/";

  async getMovie(id: string): Promise<Movie> {
    return this.get<Movie>(`movies/${encodeURIComponent(id)}`);
  }

  async getMostViewedMovies(limit = "10"): Promise<Movie[]> {
    const data = await this.get("movies", {
      params: {
        per_page: limit.toString(), // all params entries should be strings,
        order_by: "most_viewed",
      },
    });
    return data.results;
  }
}
```

RESTDataSource 클래스를 확장하여 리졸버에 필요한 모든 데이터를 가져오는 메소드를 구현할 수 있다. 이 메소드들은 내부적으로 편의상 구현되어있는 HTTP 요청수행을 위한 get, post를 사용하여야한다. 이 내부 메소드들은 query parameters를 더할 수 있고, JSON 결과를 파싱하고 캐싱할 수 있고, 중복요청을 막고, 에러를 핸들링할 수 있다. 더 복잡한 사용사례는 fetch 메소드를 바로 사용할 수도 있다. fetch 메소드는 파싱된 바디와 응답한 객체 둘다 리턴한다. 이를 사용하면 응답 헤더를 읽는 것과 같은 사용사례에 더 많은 유연성을 제공할 수 있다.

### Adding data sources to your server's context function

data sources는 context 초기함수에 더할 수 있다.

- 모든 operation에서 RESTDataSources 서브클래스의 새로운 인스턴스를 포함한 객체를 리턴한다.
- context 함수는 각 operation 마다 각각 RESTDataSource 서브클래스의 인스턴스가 생성된다.

### Caching

RESTDataSource 클래스는 두 가지 캐싱 레이어를 서브클래스에 제공한다.

- 첫번째 레이어는 기본적으로 중복적인 동시 GET 요청을 제거한다. 중복제거된 것들은 요청 메소드와 URL에 키가 입력된다. 이 동작은 `requestDeduplicationPolicyFor` 메소드를 통해 재구성할 수 있다.
- 두번째 레이어는 HTTP 캐싱 헤더를 지정하는 HTTP 응답의 결과를 캐시한다.

### GET (and HEAD) requests and responses

RESTDataSource 서브클래스를 인스턴스화할 때맏, 내부적으로 해당 인스턴스는 내부 캐시를 생성한다. 기본적으로 RESTDataSource는 내부적인 캐시에서 자동으로 결과와 함께 중복요청을 제거한다. 이 것을 중복요청제거라고한다. `requestDeduplicationPolicyFor` 메소드를 통해 이 동작을 재정의할 수 있다.

요청중복제거 캐시는 동일한 정보를 얻으려는 다른 리졸버의 중복된 GET 요청을 제거해서 현재 작업을 최적화할 수 있도록 RESTDataSource를 지원한다. 이는 DataLoader의 캐싱기능과 유사하게 작동한다.

기본적으로 쿼리는 고전적인 N+1 문제가 일어나기 쉬운데, 이 상황을 RESTDataSource가 메모화된 GET 요청과 응답의 캐시를 사용해서 작업을 최적화할 수 있다.

어떤식으로 최적화를 하냐면....

우선 처음으로 Get 요청할 때, 요청을 하기 전에 요청의 URL을 저장한다. 그런 다음 RESTDataSource는 요청을 수행하고, 그 결과를 URL과 함께 메모화된 캐시에 영구저장한다.

현재 operation의 리졸버가 동일한 URL에 대한 병렬 GET 요청 시도시, RESTDataSource는 해당 요청을 수행하기 전에 메모화된 캐시를 먼저 확인한다. 해당 요청이나 결과가 캐시에 있으면 다른 요청을 하지 않고 저장된 결과를 리턴한다.

**이런 내부 캐싱 매커니즘이 모든 요청에 대한 RESTDataSource 인스턴스를 생성하는 이유이다. 그렇지 않으면, 응답이 캐시되지 않아야한다고 명시되어있어도 요청 전체에 걸쳐 응답이 캐싱되기 때문이다.**

`cacheKeyFor` 메소드를 덮으쓰는 경우, GET 요청이 RESTDataSource의 중복제거 캐시에 저장되는 방식을 변경할 수 있다. 기본적으로는 요청의 캐시 키는 HTTP 메소드와 URL의 조합으로 구성되어있다.

```typescript
class MoviesAPI extends RESTDataSource {
  // RESTDataSource v5 이전 버전에서 중복 제거 정책을 복원하려면 다음과 같이 requestDeduplicationPolicyFor를 구성하면 된다.
  protected override requestDeduplicationPolicyFor(
    url: URL,
    request: RequestOptions
  ) {
    const cacheKey = this.cacheKeyFor(url, request);
    return {
      policy: "deduplicate-until-invalidated",
      deduplicationKey: `${request.method ?? "GET"} ${cacheKey}`,
    };
  }

  // 요청 중복 제거를 완전히 비활성화하려면 다음과 같이 requestDeduplicationPolicyFor를 구성하면 된다.
  protected override requestDeduplicationPolicyFor(
    url: URL,
    request: RequestOptions
  ) {
    const cacheKey = this.cacheKeyFor(url, request);
    return { policy: "do-not-deduplicate" } as const;
  }
}
```

### Specifying cache TTL

- 요청 메소드가 GET일 때, 캐싱 헤더를 지정해줄 수 있다.
- `RESTDataSource` 인스턴스의 `cacheOptions`를 통해 TTL을 지정할 수 있다.
  - `cacheOptionsFor` 메소드로 재정의하거나 HTTP 메소드를 통해 할 수 있다.

RESTDataSource는 캐시된 정보가 캐싱헤더에 설정된 TTL 규칙을 따르도록 한다.

각 RESTDataSource 서브클래스는 캐시인수를 받아서 과거에 fetch한 결과를 저장하는데 사용하는 캐시를 지정할 수 있다.

```typescript
// KeyValueCache is the type of Apollo server's default cache
import type { KeyValueCache } from "@apollo/utils.keyvaluecache";

class PersonalizationAPI extends RESTDataSource {
  override baseURL = "https://person.example.com/";
  private token: string;

  constructor(options: { cache: KeyValueCache; token: string }) {
    super(options); // this sends our server's `cache` through
    this.token = options.token;
  }
}

// server set up, etc.

const { url } = await startStandaloneServer(server, {
  context: async ({ req }) => {
    const token = getTokenFromRequest(req);
    // We'll take Apollo Server's cache
    // and pass it to each of our data sources
    const { cache } = server;
    return {
      dataSources: {
        moviesAPI: new MoviesAPI({ cache, token }),
        personalizationAPI: new PersonalizationAPI({ cache }),
      },
    };
  },
});
```

여러 RESTDataSource 서브클래스 인스턴스에 같은 캐시를 전달해서 인스턴스가 캐시된 결과를 공유하는 것이 가능하다.

서버의 여러 인스턴스를 실행할 때, 외부에서 공유되는 cache backend를 사용해야한다. 그러면 한 서버 인스턴스가 다른 인스턴스의 캐시된 결과를 사용할 수 있다.

### HTTP Methods

RESTDataSource는 일반적인 REST API 요청 메소드를 사용할 수 있는 편의성 메소드가 포함되어있다.

### Method parameters

모든 편의성 HTTP 메소드는 첫번째 파라미터는 엔드포인트의 상대경로이고, 두번째 파라미터는 요청의 헤더, params, cacheOptions, body 등을 셋업할 수 있다.

```typescript
class MoviesAPI extends RESTDataSource {
  override baseURL = "https://movies-api.example.com/";

  // an example making an HTTP POST request
  async postMovie(movie) {
    return this.post(
      `movies`, // path
      { body: movie } // request body
    );
  }
}
```

## Setting fetch options

두번쨰 파라미터는 각 REST 메소드가 요청 options을 포함한 객체로 넘길 수 있다. 이 options은 일반적으로 method, headers, body, signal을 포함할 수 있다.

### Setting timeouts

fetch timeout을 세팅하는 방법은 signal option을 통해서 커스텀 로직으로 요청을 중단할 수 있는 AbortSignal을 제공한다.

```typescript
this.get("/movies/1", { signal: AbortSignal.timeout(myTimeoutMilliseconds) });
```

## Intercepting fetches

Apollo Server 4에서 `@apollo/utils.fetcher`를 사용해서 fetching을 수행한다. 이 인터페이스를 사용하면 Fetch APi를 자체적으로 구현할 수 있다. 모든 Fetch 구현과의 호환성을 보장하기 위해 `willSendRequest`와 같은 제공되는 훅스는 Request 객체가 아니라 일반 JS 객체이다.

`RESTDataSource`에는 요청이 전송되기전에 재정의할 수 있는 `willSendRequest` 메소드가 있다. 이 메소드를 사용하여 헤더 또는 쿼리 파라미터를 추가할 수 있다. 이 메소드는 모든 요청이 전송되는데에 적용되는 권한이나 기타 관련된 부분에 일반적으로 사용된다.

또한 데이터소스는 사용자 토큰 등의 관련된 정보를 저장하는데에 유용한 GraphQL operation context에 접근할 수 있다.

### Setting a header, query parameter

```typescript
import { RESTDataSource, AugmentedRequest } from "@apollo/datasource-rest";
import type { KeyValueCache } from "@apollo/utils.keyvaluecache";

class PersonalizationAPI extends RESTDataSource {
  override baseURL = "https://movies-api.example.com/";
  private token: string;

  constructor(options: { token: string; cache: KeyValueCache }) {
    super(options);
    this.token = options.token;
  }

  override willSendRequest(_path: string, request: AugmentedRequest) {
    request.headers["authorization"] = this.token;
    request.params.set("api_key", this.token);
  }
}
```

## Resolving URLs dynamically

경우에 따라 환경이나 기타 contextValues에 따라서 url을 설정하고 싶을때, `resolveURL`을 재정의한다. -> 무슨뜻? 아 환경에 따라서 baseUrl이 바뀔때

```typescript
import { RESTDataSource, AugmentedRequest } from "@apollo/datasource-rest";
import type { KeyValueCache } from "@apollo/utils.keyvaluecache";

class PersonalizationAPI extends RESTDataSource {
  private token: string;

  constructor(options: { token: string; cache: KeyValueCache }) {
    super(options);
    this.token = options.token;
  }

  override async resolveURL(path: string, request: AugmentedRequest) {
    if (!this.baseURL) {
      const addresses = await resolveSrv(
        path.split("/")[1] + ".service.consul"
      );
      this.baseURL = addresses[0];
    }
    return super.resolveURL(path, request);
  }
}
```
