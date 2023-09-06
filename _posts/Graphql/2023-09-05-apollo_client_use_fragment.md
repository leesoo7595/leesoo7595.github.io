---
layout: post
title: "Apollo Client - useFragment"
date: 2023-09-05
tags: [Graphql]
comments: true
---

## `useFragment`

`useFragment` 훅은 아폴로 클라이언트 캐시에서 lightweight live binding을 나타낸다.. 이것은 아폴로 클라이언트에서 특정 fragment 결과를 각각의 컴포넌트에 broadcast하게 사용할 수 있다. 이 훅은 캐시가 현재 주어진 프래그먼트에 포함되어있는 모든 최신 데이터(always-up-to-date)를 리턴한다. `useFragment`는 이것만으로는 네트워크 요청을 발생시키지 않는다.

`useQuery` 훅은 캐시에 있는 데이터를 쿼리하고 채우는 기본 훅이다. 결과적으로 `useFragment`를 통한 프래그먼트 데이터를 읽는 컴포넌트는 여전히 쿼리 데이터의 모든 변경사항에 대해 구독하지만, 해당 프래그먼트의 특정 데이터가 변경될 때만 업데이트를 받는다.

### 예시

```javascript
const ItemFragment = gql`
  fragment ItemFragment on Item {
    text
  }
`;
```

`useQuery` 훅을 사용하여 ListQuery의 list 안에 ItemFragment를 스프레딩하여 기재되어있는 text 필드 뿐만 아니라 id가 있는 리스트들을 받아올 수 있다.

```javascript
const listQuery = gql`
  query GetItemList {
    list {
      id
      ...ItemFragment
    }
  }
  ${ItemFragment}
`;

function List() {
  const { loading, data } = useQuery(listQuery);

  return (
    <ol>
      {data?.list.map((item) => (
        <Item key={item.id} id={item.id} />
      ))}
    </ol>
  );
}
```

> 각 쿼리 도큐먼트의 프래그먼트 대신, 아폴로 클라이언트의 `createFragmentRegistry` 메소드를 사용하여 `InMemoryCache` 안에 프래그먼트의 이름을 먼저 등록할 수 있다. 이것은 아폴로 클라이언트가 네트워크 요청이 가기전에 document에 등록된 프래그먼트에 대한 정의를 포함할 수 있다. 자세한 내용은 `createFragmentRegistry` 사용하기 보기

`<Item>` 컴포넌트의 `useFragment`를 사용하여 `fragment` document, `fragmentName`과 `from`을 통한 객체 참조를 통해 각 아이템에 대한 live binding을 생성할 수 있다.

```javascript
function Item(props: { id: number }) {
  const { complete, data } = useFragment({
    fragment: ItemFragment,
    fragmentName: "ItemFragment",
    from: {
      __typename: "Item",
      id: props.id,
    },
  });

  return <li>{complete ? data.text : "incomplete"}</li>;
}
```

> `useFragment`는 모든 리스트를 다시 렌더링하지 않고 각 아이템들이 캐시 업데이트에 반응해야하는 경우, `@nonreactive` 디렉티브와 함께 사용할 수 있다.

## `@nonreactive`

`@nonreactive` 디렉티브는 쿼리의 필드나 프래그먼트를 표시하는데 사용할 수 있고, 이 디렉티브가 표시된 하위 트리에 포함된 데이터의 변경으로 인해 렌더링이 발생하지않아야한다는 것을 나타낼 때 사용한다. 이를 통해 상위 컴포넌트는 `@nonreactive`으로 표시된 필드에 해당하는 데이터가 변경될 때 다시 렌더링하지 않고도 자식 컴포넌트가 렌더링할 데이터를 가져올 수 있다.

스키 코스 리스트를 가져와 렌더링하는 App 컴포넌트를 생각해보자:

```javascript
const TrailFragment = gql`
  fragment TrailFragment on Trail {
    name
    status
  }
`;

const ALL_TRAILS = gql`
  query allTrails {
    allTrails {
      id
      ...TrailFragment @nonreactive
    }
  }
  ${TrailFragment}
`;

function App() {
  const { data, loading } = useQuery(ALL_TRAILS);
  return (
    <main>
      <h2>Ski Trails</h2>
      <ul>
        {data?.trails.map((trail) => (
          <Trail key={trail.id} id={trail.id} />
        ))}
      </ul>
    </main>
  );
}
```

Trail 컴포넌트는 trail의 name과 status를 렌더링하고 mutation을 실행하여 trail의 status를 `OPEN`과 `CLOSED`로 바꿀 수 있다:

```javascript
const Trail = ({ id }) => {
  const [updateTrail] = useMutation(UPDATE_TRAIL);
  const { data } = useFragment({
    fragment: TrailFragment,
    from: {
      __typename: "Trail",
      id,
    },
  });
  return (
    <li key={id}>
      {data.name} - {data.status}
      <input
        checked={data.status === "OPEN" ? true : false}
        type="checkbox"
        onChange={(e) => {
          updateTrail({
            variables: {
              trailId: id,
              status: e.target.checked ? "OPEN" : "CLOSED",
            },
          });
        }}
      />
    </li>
  );
};
```

Trail 컴포넌트는 props를 통해서 전체 Trail 객체를 반아오는 것이 아니라, 캐시에 있는 각 Trail 데이터에 대한 live binding을 생성하기 위해 fragment document와 함꼐 사용되는 ID만 받아온다는 것에 주의하자. 이렇게 하면 Trail 컴포넌트가 각 Trail 캐시 업데이트에 독립적으로 반응할 수 있다. @nonreactive 디렉티브를 TrailFragment에 적용시켜놓았기 때문에, status를 업데이트 하는 것이 부모의 App 컴포넌트를 리렌더하도록 만들지 않는다.
