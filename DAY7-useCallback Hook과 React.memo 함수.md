### useCallback Hook

함수를 위한 Hook인 useCallback Hook 함수를 사용해서 이전에 만들었던 함수를 새로 만들지 않고 재사용하는 방법에 대해 알아보자.

우리가 만든 함수는 컴포넌트가 리렌더링되면 재선언돼서 다시 새롭게 만들어진다. 함수를 새로 만드는 것 자체가 메모리도 CPU도 리소스를 많이 차지하지는 않지만 한번 만든 함수를 재사용하는 것이 좋지만은 않은 이유가 나중에 컴포넌트들이 props가 바뀌지 않았을 때 가상돔에 리렌더링되지 않도록 최적화 작업을 해줄 텐데 이때 우리가 만든 함수가 재사용되지 않고 매번 새롭게 만들어지는 경우 최적화가 제대로 이루어지지 않는다. 따라서 useCallback을 사용해서 함수를 재사용할 필요가 있다.



```react
import React, { useRef, useState, useMemo, useCallback } from "react";//우선 useCallback을 불러와줌
import CreateUser from "./CreateUser";
import UserList from "./UserList";

function countActiveUsers(users) {
  console.log("Active:true인 활성 사용자 수를 세는중...");
  return users.filter((user) => user.active).length;
}

function App() {
  const [inputs, setInputs] = useState({
    username: "",
    email: "",
  });

  const { username, email } = inputs;

    //useCallback으로 우리가 만든 함수를 감싸준다.
    
  const onChange = useCallback(
    (e) => {
      const { name, value } = e.target;
      setInputs({
        ...inputs,
        [name]: value,
      });
    },
    [inputs]
  ); //onChange함수는 inputs가 바뀔때만 함수가 새로 만들어지고 그렇지 않은 경우 기존에 만든 함수를 재사용한다.

  const [users, setUsers] = useState([
    {
      id: 1,
      username: "test1",
      email: "public.velopert1@gmail.com",
      active: true,
    },
    {
      id: 2,
      username: "test2",
      email: "public.velopert2@gmail.com",
      active: false,
    },
    {
      id: 3,
      username: "test3",
      email: "public.velopert3@gmail.com",
      active: false,
    },
  ]);

  const nextId = useRef(4);

  const onCreate = useCallback(() => {
    const user = {
      id: nextId.current,
      username,
      email,
    };
    setUsers([...users, user]);
    setInputs({
      username: "",
      email: "",
    });
    nextId.current += 1;
  }, [username, email, users]);
  //useCallback 내부에서 참조하고 있는 상태, 컴포넌트에서 props로 받아온 값이 있다하면 꼭 deps에 넣어줄것
  //username,email,users 모두 상태이기 때문에 생략하면 이 onCreate함수 내부에서 이 상태들을 참조할때 최신상태를 참조하지 않고 이전에 컴포넌트가 만들어질때 상태를 참조한다.

  const onRemove = useCallback(
    (id) => {
      setUsers(users.filter((user) => user.id !== id));
    },
    [users]
  );

  const onToggle = useCallback(
    (id) => {
      setUsers(
        users.map((user) =>
          user.id === id ? { ...user, active: !user.active } : user
        )
      );
    },
    [users]
  );
  
  const count = useMemo(() => countActiveUsers(users), [users]); 
  return (
    <>
      <CreateUser
        username={username}
        email={email}
        onChange={onChange}
        onCreate={onCreate}
      />
      <UserList users={users} onRemove={onRemove} onToggle={onToggle} />
      <div>활성 사용자 수 :{count}</div>
    </>
  );
}

export default App;

```

App 컴포넌트 안에 만든 함수들을 모두 useCallback으로 감싸주고 deps의 값으로 참조하고 있는 상태를 넣어주었다.



### React.memo

컴포넌트 리렌더링 성능을 최적화할 수 있는 React.memo라는 함수에 대해서 알아보자. 이 함수는 컴포넌트에서 리렌더링이 불필요할 때는 이전에 렌더링 했던 결과를 재사용할 수 있도록 해준다.



사용법은 간단한데 바로 컴포넌트를 내보낼때 컴포넌트를 React.memo로 감싸 주는 것이다.
(컴포넌트 자체를 React.memo로 감싸주어도 되지만 내보낼때 최적화를 해줘도 됨- 가독성 측면에서 좀 더 좋다)

```react
export default React.memo(CreateUser);
export default React.memo(UserList);
```

그리고 export하고 있지 않은 User 컴포넌트는 컴포넌트 자체를  React.memo로 감싸주면 된다.

```react
const User = React.memo(function User({ user, onRemove, onToggle }) {
  useEffect(() => {
    console.log(user);
  }); //user가 설정(mount될때!)되거나 업데이트가 되도 동작하게됨

  return (
    <div>
      <b
        style={{
          color: user.active ? "green" : "black",
          cursor: "pointer",
        }}
        onClick={() => onToggle(user.id)}
      >
        {user.username}
      </b>
      &nbsp;
      <span>({user.email})</span>
      <button
        onClick={() => {
          onRemove(user.id);
          // console.log(user);
        }}
      >
        삭제
      </button>
    </div>
  );
});
```



근데 여기서 끝난것이 아니다. 지금까지 구현한 최적화로는 컴포넌트 렌더링에대한 최적화가 완벽하지 않은 상태이기때문이다.
우리는 App.js에서 최적화하기위해 useCallback을 사용하고 deps를 users로 설정해주었는데 이렇게 하면 users가 바뀔 때마다 onCreate,onRemove,onToggle가 새로 만들어지고 그것을 props로 받고있는 UserList가 리렌더링 되면서 User 컴포넌트 전체가 리렌더링된다. React.memo를 통해 props가 변경되면 리렌더링되도록했는데 users가 바뀌면 컴포넌트가 받은 props가 바뀌니 리렌더링되서 문제가 된것이다.따라서 이를 방지하기위한 최적화가 추가로 필요하다.



이를 해결하기 위한 방법은 App.js의 함수에서 useState의 함수형 업데이트를 하는 것이다. 

```react
const onCreate = useCallback(() => {
    const user = {
      id: nextId.current,
      username,
      email,
    };
    setUsers((users) => users.concat(user));
    //이렇게 함수형 업데이트를 하면 파라미터로 받은 users에서 최신상태를 조회해주고 있기 때문에 
    //deps에 users를 빼주어도된다.
    // 그러면 onCreate에서만 deps를 [username, email]로 하고 나머지는 []로 해두면된다.
    
    setInputs({
      username: "",
      email: "",
    });
    nextId.current += 1;
  }, [username, email]);
//onCreate는 username,email이 바뀔때에만 새로 만들어짐 



  const onRemove = useCallback((id) => {
    setUsers((users) => users.filter((user) => user.id !== id));
  }, []);
//onRemove는 컴포넌트가 처음 렌더링될때 딱 한번 만들어짐



  const onToggle = useCallback((id) => {
    setUsers((users) =>
      users.map((user) =>
        user.id === id ? { ...user, active: !user.active } : user
      )
    );
  }, []);
//onToggle 는 컴포넌트가 처음 렌더링될때 딱 한번 만들어짐 
```

이렇게 하면 컴포넌트가 만들어질때 딱 한번만 선언해서 만들어지고 그 이후에는 함수를 재사용해서 사용할수 있게된다.