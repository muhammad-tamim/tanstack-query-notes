<h1 align="center">Tanstack Query Notes</h1>

- [Introduction:](#introduction)
    - [Installation:](#installation)
    - [Why Use TanStack Query?](#why-use-tanstack-query)

# Introduction:
TanStack Query is a server-state management library. It can handle data fetching, caching, background refetching, loading & error states, pagination & infinite scrolling, mutations and more. 

**Note:** It can completely replace manual useEffect and useState for data fetching in react applications.

### Installation: 

```bash
npm i @tanstack/react-query
```

### Why Use TanStack Query?

- without fetch + useState, and useEffect: 

```js
import { useEffect, useState } from "react";

function App() {
  const [users, setUsers] = useState([]);
  const [pending, setPending] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/use")
      .then((res) => res.json())
      .then((data) => {
        setUsers(data);
        setPending(false);
      })
      .catch((err) => {
        setError(err.message);
        setPending(false);
      });
  }, []);

  if (pending) {
    return <h2 className="text-center text-5xl">Loading......</h2>;
  }

  if (error) {
    return <h2 className="text-red-500">Error: {error}</h2>;
  }

  return (
    <div>
      {users.map((user) => <div key={user.id}>
        <p>{user.name} | {user.email}</p>
      </div>)}
    </div >
  );
}

export default App;
```

![alt text](./assets/images/fetch-useState-useEffect-v1.png)

Note: this version don't catch fetch error properly becaus fetch only throw error for network failure, not for 4xx or 5xx response status. for that type of error handling we need to check response.ok property. 

```js
import { useEffect, useState } from "react";

function App() {
  const [users, setUsers] = useState([]);
  const [pending, setPending] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch("https://jsodfdfdfdfdfdfder.typicode.com/use")
      .then((res) => {
        if (!res.ok) {
          throw new Error(`Request failed with status ${res.status}`);
        }
        return res.json();
      })
      .then((data) => {
        setUsers(data);
        setPending(false);
      })
      .catch((err) => {
        setError(err.message);
        setPending(false);
      });
  }, []);

  if (pending) {
    return <h2 className="text-center text-5xl">Loading......</h2>;
  }

  if (error) {
    return <h2 className="text-red-500">Error: {error}</h2>;
  }

  return (
    <div>
      {users.map((user) => (
        <div key={user.id}>
          <p>{user.name} | {user.email}</p>
        </div>
      ))}
    </div>
  );
}

export default App;
```

![alt text](./assets/images/fetch-useState-useEffect-v2.png)

- with axios + useState, and useEffect: 

```js
import { useEffect, useState } from "react";
import axios from "axios";

function App() {
  const [users, setUsers] = useState([]);
  const [pending, setPending] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    axios.get("https://wrongURl.com/users")
      .then((res) => {
        setUsers(res.data);
        setPending(false);
      })
      .catch((err) => {
        setError(err.message);
        setPending(false);
      });
  }, []);

  if (pending) {
    return <h2 className="text-center text-5xl">Loading......</h2>;
  }

  if (error) {
    return <h2 className="text-red-500">Error: {error}</h2>;
  }

  return (
    <div>
      {users.map((user) => (
        <div key={user.id}>
          <p>
            {user.name} | {user.email}
          </p>
        </div>
      ))}
    </div>
  );
}

export default App;
```

![alt text](./assets/images/fetch-useState-useEffect-v2.png)

Note: This axios version is better than fetch version because axios throw error for 4xx and 5xx response status by default.

- with Tanstack Query + axios: 

```js
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import './index.css'
import App from './App.jsx'

createRoot(document.getElementById('root')).render(
  
  <StrictMode>
    <QueryClientProvider client={queryClient}>
        <App />
    </QueryClientProvider>  
  </StrictMode>,
)
```

```js
import axios from "axios";
import { useQuery } from "@tanstack/react-query";

const fetchUsers = async () => {
  const res = await axios.get("https://.typicode.com/users");
  return res.data;
};

function App() {
  const { data: users, isLoading, isError, error } = useQuery({
    queryKey: ["users"],
    queryFn: fetchUsers,
  });

  if (isLoading) {
    return <h2 className="text-center text-5xl">Loading......</h2>;
  }

  if (isError) {
    return <h2 className="text-red-500">Error: {error.message}</h2>
  }

  return (
    <div>
      {users.map((user) => <div key={user.id}>
        <p>{user.name} | {user.email}</p>
      </div>
      )}
    </div>
  );
}

export default App;
```

![alt text](./assets/images/fetch-useState-useEffect-v2.png)

Note: tanstack query is really advance, if we put wrong url in the axios get method, it will automatically retry 3 times before throwing error. 


fix url version: 

```js
import axios from "axios";
import { useQuery } from "@tanstack/react-query";

const fetchUsers = async () => {
  const res = await axios.get("https://jsonplaceholder.typicode.com/users");
  return res.data;
};

function App() {
  const { data: users, isLoading, isError, error } = useQuery({
    queryKey: ["users"],
    queryFn: fetchUsers,
  });

  if (isLoading) {
    return <h2 className="text-center text-5xl">Loading......</h2>;
  }

  if (isError) {
    return <h2 className="text-red-500">Error: {error.message}</h2>
  }

  return (
    <div>
      {users.map((user) => <div key={user.id}>
        <p>{user.name} | {user.email}</p>
      </div>
      )}
    </div>
  );
}

export default App;
```
![alt text](./assets/images/tanstackQuery-axios.png)

here, we can now see tanstack query by default handle loading, error and data states. we don't need to manually create useState and useEffect for that.

