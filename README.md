# reason-apollo-hooks

Reason bindings for https://github.com/trojanowski/react-apollo-hooks

## Installation

```
yarn add reason-apollo-hooks reason-apollo
```

Follow the installation instructions of [https://github.com/mhallin/graphql_ppx](graphql_ppx).

Then update your bsconfig.json

```diff
"bs-dependencies": [
  ...
+ "reason-apollo-hooks",
+ "reason-apollo"
]
```

## Setting up 
Add the provider in the top of the tree

```reason
/* Create an InMemoryCache */
let inMemoryCache = ApolloInMemoryCache.createInMemoryCache();

/* Create an HTTP Link */
let httpLink =
  ApolloLinks.createHttpLink(~uri="http://localhost:3010/graphql", ());

let client =
  ReasonApollo.createApolloClient(~link=httpLink, ~cache=inMemoryCache, ());
  
let app = 
 <ReasonApolloHooks.ApolloProvider client>
   ...
 </ReasonApolloHooks.ApolloProvider>
```

# Available hooks

## useQuery

```reason
module UserQueryConfig = [%graphql {|
  query UserQuery {
    currentUser {
      name
    }
  }
|}];

module UserQuery = ReasonApolloHooks.Query.Make(UserQueryConfig);

[@react.component]
let make = () => {
  /* Both variant and records available */
  let (simple, _full) = UserQuery.use(());

  <div>
  {
    switch(simple) {
      | Loading => <p>{React.string("Loading...")}</p>
      | Data(data) =>
        <p>{React.string(data##currentUser##name)}</p>
      | NoData
      | Error(_) => <p>{React.string("Get off my lawn!")}</p>
    }
  }
  </div>
}
```
Using the `full` record for more advanced cases

```reason
[@react.component]
let make = () => {
  /* Both variant and records available */
  let (_simple, full) = UserQuery.use(());

  <div>
  {
    switch(full) {
      | { loading: true }=> <p>{React.string("Loading...")}</p>
      | { data: Some(data) } =>
        <p>{React.string(data##currentUser##name)}</p>
      | any other possibilities =>
      | { error: Some(_) } => <p>{React.string("Get off my lawn!")}</p>
    }
  }
  </div>
}
```


## useMutation

```reason
module ScreamMutationConfig = [%graphql {|
  mutation ScreamMutation($screamLevel: Int!) {
    scream(level: $screamLevel) {
      error
    }
  }
|}];

module ScreamMutation = ReasonApolloHooks.Mutation.Make(ScreamMutationConfig);

[@react.component]
let make = () => {
  /* Both variant and records available */
  let screamMutation = ScreamMutation.use();
  let scream = (_) => {
    screamMutation(
      ScreamMutation.options(
        ~variables=ScreamMutationConfig.make(~screamLevel=10, ())##variables,
        ()
      ),
      ()
    )
      |> Js.Promise.then_(result => {
          switch(result) {
            | Data(data) => ...
            | Error(error) => ...
            | NoData => ...
          }
          Js.Promise.resolve()
        })
      |> ignore
  } 

  <div>
    <button onClick={scream}>
      {React.string("You kids get off my lawn!")}
    </button>
  </div>
}
```

## Getting it running

```sh
npm install
npm start
```
