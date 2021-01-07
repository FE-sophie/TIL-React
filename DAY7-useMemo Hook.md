### useMemo Hook



useMemo Hook을 통해 이전에 연산된 값을 재사용하는 방법에 대해 알아보도록 하자.이 Hook함수는 성능을 최적화하기 위해 주로 사용한다.



App.js에서 다음과 같이 active 상태가 true인 활성화 사용자 수를 세는 함수를 만들고 useMemo를 사용해서 최적화를 구현해보자 

```react
import React, { useRef, useState, useMemo } from "react";
import CreateUser from "./CreateUser";
import UserList from "./UserList";

function countActiveUsers(users) {
  console.log("Active:true인 활성 사용자 수를 세는중...");
  return users.filter((user) => user.active).length; //active가 true인것만 가져와서 수를 연산해서
}

function App() {
  const [inputs, setInputs] = useState({
    username: "",
    email: "",
  });

  const { username, email } = inputs;

  const onChange = (e) => {
    const { name, value } = e.target;
    setInputs({
      ...inputs,
      [name]: value,
    });
  };
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

  const onCreate = () => {
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
  };

  const onRemove = (id) => {
    setUsers(users.filter((user) => user.id !== id));
  };
  const onToggle = (id) => {
    setUsers(
      users.map((user) =>
        user.id === id ? { ...user, active: !user.active } : user
      )
    );
  };
  
  const count = useMemo(() => countActiveUsers(users), [users]); // useMemo로 감싸준 함수는 users 상태가 바뀔때에만 호출되고 그렇지 않으면 이전에 사용한 값을 재사용한다.
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

useMemo를 사용하지 않는 경우 users의 상태가 업데이트되었을 때뿐만 아니라 onChange이벤트가 발생해서 inputs의 상태가 바뀌어도 컴포넌트가 리렌더링되면서 불필요하게 countActiveUsers가 실행된다. 

이때 사용하는 것이 useMemo Hook이다.use Memo는 특정값이 바뀌었을때만 원하는 함수를 실행해서 연산을 해주고 원하는 값이 바뀌지 않으면 리렌더링 할 때 이전에 만든 값을 재사용할 수 있도록 도와준다.

useMemo의 첫 번째 파라미터는 함수 형태여야 한다. 두 번째 파라미터는 deps 값인데  [] 배열 안에 넣는 값이 바뀌어야만 첫 번째 파라미터로 넣은 함수가 호출되게 되는 것이다.

이렇게 useMemo를 사용하면 우리가 정말 필요할 때만 연산을 할 수 있게 해주기 때문에 컴포넌트 최적화를 할 때 사용할 수 있다.

