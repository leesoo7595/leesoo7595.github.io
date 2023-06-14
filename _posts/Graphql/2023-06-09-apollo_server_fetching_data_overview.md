---
layout: post
title: "Apollo Server - Fetching Data"
date: 2023-06-10
tags: [Graphql]
comments: true
---

# Fetching Data

## Overview

데이터베이스 및 datasources 관련 커넥션 관리

아폴로 서버는 REST API 또는 데이터베이스에서 필요한 모든 소스를 fetch 해올 수 있다. 아폴로 서버에서는 여러가지 다양한 데이터소스를 사용할 수 있다.

<img width="728" alt="스크린샷 2023-06-14 오전 2 12 27" src="https://github.com/leesoo7595/leesoo7595.github.io/assets/39291812/6f0087f4-8e5e-4b34-955b-e6102db1562e">

서버에서 다양한 데이터소스를 사용할 수 있어서, resolvers를 깔끔하게 유지하는 것이 더 중요해졌다.

그래서 특정 데이터소스에서 데이터를 가져오는 로직을 캡슐화한 개별 데이터소스 클래스를 만드는 것을 추천한다. 그 클래스는 resolver가 데이터에 깔끔하게 접근하는데 사용할 수 있는 메서드를 제공하여야 한다. 또한 리졸빙하는 동안 데이터소스 클래스를 커스텀하여 캐싱, 중복제거, 에러를 지원할 수 있다.

### Creating data source classes

데이터소스 클래스는 필요에 따라 간단하거나 복잡할 수 있다. 서버가 어떤 데이터를 필요로 하는지 알고 그 뎅이터를 사용하면 클래스에 포함된 메소드에 대한 가이드가 될 수 있다.

아래는 예약을 저장하는 데이터베이스에 연결하는 데이터소스 클래스의 예시이다.

```typescript
export class ReservationsDataSource {
  private dbConnection;
  private token;
  private user;

  constructor(options: { token: string }) {
    this.dbConnection = this.initializeDBConnection();
    this.token = options.token;
  }

  async initializeDBConnection() {
    // set up our database details, instantiate our connection,
    // and return that database connection
    return dbConnection;
  }

  async getUser() {
    if (!this.user) {
      // store the user, lookup by token
      this.user = await this.dbConnection.User.findByToken(this.token);
    }
    return this.user;
  }

  async getReservation(reservationId) {
    const user = await this.getUser();
    if (user) {
      return await this.dbConnection.Reservation.findByPk(reservationId);
    } else {
      // handle invalid user
    }
  }

  //... more methods for finding and creating reservations
}
```

### Batching and caching

배치, 중복, 캐시를 데이터소스 클래스에 추가하고 싶다면, DataLoader 패키지를 사용할 것을 추천한다. 이와 같은 패키지를 사용하면 n+1 쿼리 문제를 해결하는데 유용하다.

DataLoader는 메모화 캐시를 제공하여 한 Graphql 요청에서 동일한 쿼리를 여러번 로드하는 것을 방지한다. (RESTDataSource의 캐싱 레이어중 하나와 비슷) 또한 이벤트 루프의 한 tick 동안, 여러 객체를 한번에 가져오는 배치 요청으로 조합한다.

DataLoader 인스턴스는 요청별로 생성되어서 데이터소스에서 데이터로더를 사용하는 경우, 요청할 때마다 해당 클래스의 인스턴스를 새로 만들어야한다.

```typescript
import DataLoader from "dataloader";

class ProductsDataSource {
  private dbConnection;

  constructor(dbConnection) {
    this.dbConnection = dbConnection;
  }

  private batchProducts = new DataLoader(async (ids) => {
    const productList = await this.dbConnection.fetchAllKeys(ids);
    // Dataloader expects you to return a list with the results ordered just like the list in the arguments were
    // Since the database might return the results in a different order the following code sorts the results accordingly
    const productIdToProductMap = productList.reduce((mapping, product) => {
      mapping[product.id] = product;
      return mapping;
    }, {});
    return ids.map((id) => productIdToProductMap[id]);
  });

  async getProductFor(id) {
    return this.batchProducts.load(id);
  }
}

// In your server file

// Set up our database, instantiate our connection,
// and return that database connection
const dbConnection = initializeDBConnection();

const { url } = await startStandaloneServer(server, {
  context: async () => {
    return {
      dataSources: {
        // Create a new instance of our data source for every request!
        // (We pass in the database connection because we don't need
        // a new connection for every request.)
        productsDb: new ProductsDataSource(dbConnection),
      },
    };
  },
});
```

### Adding data sources to your context function

서버의 context 초기함수로 data sources를 추가할 수 있다.

```typescript
interface ContextValue {
  dataSources: {
    dogsDB: DogsDataSource;
    catsApi: CatsAPI;
  };
  token: string;
}

const server = new ApolloServer<ContextValue>({
  typeDefs,
  resolvers,
});

const { url } = await startStandaloneServer(server, {
  context: async () => {
    const { cache } = server;
    const token = req.headers.token;
    return {
      // We create new instances of our data sources with each request.
      // We can pass in our server's cache, contextValue, or any other
      // info our data sources require.
      dataSources: {
        dogsDB: new DogsDataSource({ cache, token }),
        catsApi: new CatsAPI({ cache }),
      },
      token,
    };
  },
});
```

이렇게 하면 아폴로 서버는 모든 operation에서 context 초기화 함수를 호출할 것이다.

- 모든 operation에서 context는 데이터소스 클래스의 인스턴스를 포함한 객체를 리턴한다.
- 데이터소스가 stateful하다면, context 함수는 각 operation 마다 새로운 데이터소스 클래스의 인스턴스를 만들어야한다. 그래야 데이터소스에서 비효율적으로 요청을 여러번하는 일이 발생하지 않는다.

이렇게 하면 resolvers에서 contextValue를 통해 데이터소스에 접근하여 데이터를 가져올 수 있다.

```typescript
const resolvers = {
  Query: {
    dog: async (_, { id }, { dataSources }) => {
      return dataSources.dogsDB.getDog(id);
    },
    popularDogs: async (_, __, { dataSources }) => {
      return dataSources.dogsDB.getMostLikedDogs();
    },
    bigCats: async (_, __, { dataSources }) => {
      return dataSources.catsApi.getCats({ size: 10 });
    },
  },
};
```

### Open-source implementations

Apollo Server 3에서는 서브클래스를 가질 수 있는 data sources가 있는 DataSource라는 추상클래스가 포함되어있다. 데이터 원본을 백그라운드의 context에 연결하는 `dataSources` 함수를 사용해서 DataSource 서브클래스를 초기화할 수 있다.

Apollo Server 4에서는 DataSource 수퍼클래스를 전적으로 피해서 각 요청시 같은 context 함수를 사용하여 data sources를 만들 수 있다. 우리는 각각 특정한 source에 맞는 클래스로 data source의 커스텀 클래스를 만드는 것을 추천한다.

### Modern data sources

Apollo는 Apollo Server 4에서 RESTDataSource 클래스를 오픈소스로 관리한다.

커뮤니티의 경우 BatchedSQLDataSource라는 오픈소스를 관리하고 있다.

### Legacy data sources classes

아래의 데이터소스 구현은 더 이상 사용되지않는 apollo-datasource 패키지에서 일반 DataSource 추상 클래스를 확장한 것이다.

DataSource의 서브클래스는 특정 스토어, API와 통신하는 데 필요한 로직을 정의한다.

- HTTPDataSource
- SQLDataSource
- MongoDataSource
- CosmosDataSource
- FirestoreDataSource

  아폴로에서는 커뮤니티에서 관리하는 라이브러리에 대한 공식적인 지원은 하지않는다. 따라서 커뮤니티에서 관리하는 라이브러리가 모범 사례를 준수하거나, 이가 계속 관리될 것이라고 보장할 수 없다.

#### Using DataSource subclasses

apollo server 3에서는 각각 새로운 DataSource가 백그라운드에서 실행될때 DataSource 서브클래스를 구축한 후 즉시 initialize 메소드가 실행된다.

apollo server 4에서 이를 복제하려면, DataSource 서브클래스의 생성자 함수에서 초기화 메소드를 수동으로 호출할 수 있다.

```typescript
import { ApolloServer } from "@apollo/server";
import { startStandaloneServer } from "@apollo/server/standalone";
import { KeyValueCache } from "@apollo/utils.keyvaluecache";
import { Pool } from "undici";
import { HTTPDataSource } from "apollo-datasource-http";

class MoviesAPI extends HTTPDataSource {
  override baseURL = "https://movies-api.example.com/";

  constructor(options: { cache: KeyValueCache<string>; token: string }) {
    // the necessary arguments for HTTPDataSource
    const pool = new Pool(baseURL);
    super(baseURL, { pool });

    // We need to call the initialize method in our data source's
    // constructor, passing in our cache and contextValue.
    this.initialize({ cache: options.cache, context: options.token });
  }

  async getMovie(id: string): Promise<Movie> {
    return this.get<Movie>(`movies/${encodeURIComponent(id)}`);
  }
}

interface MyContext {
  dataSources: {
    moviesApi: MoviesAPI;
  };
  token?: string;
}

const server = new ApolloServer<MyContext>({
  typeDefs,
  resolvers,
});

const { url } = await startStandaloneServer(server, {
  context: async () => {
    const { cache } = server;
    const token = req.headers.token;
    return {
      dataSources: {
        moviesApi: new MoviesAPI({ cache, token }),
      },
      token,
    };
  },
});
```

## Reference

- [Apollo Server - Fetching Data](https://www.apollographql.com/docs/apollo-server/data/fetching-data/#creating-data-source-classes)
